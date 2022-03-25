<template>
  <div>
    <h1>You are {{ player }}</h1>
    <div style="display: flex; justify-content: center">
      <div>
        <video
          id="caller"
          autoplay
          muted
          style="width: 100px; height: 100px; border: 1px solid dodgerblue"
        ></video>
        <p>You</p>
      </div>
      <div style="margin-left: 2rem">
        <video
          id="callee"
          autoplay
          style="width: 100px; height: 100px; border: 1px solid dodgerblue"
        ></video>
        <p>Other person</p>
      </div>
    </div>
    <div v-if="currentState === 'DISCONNECTED'">
      <ion-button @click="connect">Connect</ion-button>
    </div>
    <div>
      <ion-button v-if="currentState === 'IDLE'" @click="call"
        >Call other</ion-button
      >
      <div v-else-if="currentState === 'CALLING'">
        <ion-button @click="endCall">Stop calling</ion-button>
        <p>Dialing...</p>
      </div>
      <div v-else-if="currentState === 'RECEIVING'">
        <p>Incoming call from {{ currentCall?.caller.nickname }}</p>
      </div>
      <ion-button v-else-if="currentState === 'IN_CALL'" @click="endCall"
        >End call</ion-button
      >
    </div>
    <div>
      <ion-button v-if="currentState === 'RECEIVING'" @click="acceptCall"
        >Accept call</ion-button
      >
      <ion-button v-if="currentState === 'RECEIVING'" @click="declineCall"
        >Decline call</ion-button
      >
    </div>
    <div style="display: flex; flex-direction: column; margin-top: 2rem">
      <span>{{ currentState }}</span>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, defineProps, ref } from "vue";
import SendBirdCall, { DirectCall } from "sendbird-calls";
import { IonButton } from "@ionic/vue";
type State = "DISCONNECTED" | "IDLE" | "RECEIVING" | "CALLING" | "IN_CALL";

const props = defineProps<{
  player: string;
}>();

const isMe = computed(() => props.player === "me");
const currentState = ref<State>("DISCONNECTED");
const currentCall = ref<DirectCall | null>(null);

const connect = async () => {
  // Initialize the sendbird instance with our APP_ID.
  await SendBirdCall.init("A3BA2ED9-F7A0-4CCC-9D1A-771200FC0A01");

  // Ask for audio and video permissions if not already granted
  SendBirdCall.useMedia({ audio: true, video: true });

  // Authenticate user
  const authOptions = {
    userId: isMe.value ? "p3aceful" : "friend",
    accessToken: isMe.value
      ? "6db55614400c54b26249687c00252ddbff505a68"
      : "7a030c1b57be84d071258606c9a3df5f40c6048c",
  };

  await SendBirdCall.authenticate(authOptions, (_result, error) => {
    if (error) {
      currentState.value = "DISCONNECTED";
      currentCall.value = null;
    }
  });

  await SendBirdCall.connectWebSocket()
    .then(() => {
      console.log("Successfully connected");
    })
    .catch(() => {
      console.log("Successfully NOT connected");
      currentState.value = "DISCONNECTED";
      currentCall.value = null;
    });

  currentState.value = "IDLE";

  SendBirdCall.addListener("MY_HANDLER_ID", {
    onRinging: (call) => {
      console.log("Someone is ringing", { call });

      currentState.value = "RECEIVING";
      currentCall.value = call;

      call.onEstablished = (call) => {
        currentState.value = "IN_CALL";
        currentCall.value = call;
      };

      call.onEnded = (call) => {
        currentState.value = "IDLE";
        currentCall.value = null;
      };
    },
  });
};

const call = async () => {
  const dialParams = {
    userId: isMe.value ? "friend" : "peaceful",
    isVideoCall: true,
    callOption: {
      localMediaView: document.getElementById("caller") as HTMLMediaElement,
      remoteMediaView: document.getElementById("callee") as HTMLMediaElement,
      audioEnabled: true,
      videoEnabled: true,
    },
  };

  const call = SendBirdCall.dial(dialParams, (call, error) => {
    if (error) {
      currentState.value = "IDLE";
      currentCall.value = null;
    }

    // Dialing succeeded
    currentState.value = "CALLING";
    if (call) {
      currentCall.value = call;
    }
  });

  call.onEstablished = (call) => {
    currentState.value = "IN_CALL";
  };

  //   call.onConnected = (call) => {};

  call.onEnded = (call) => {
    currentState.value = "IDLE";
    currentCall.value = null;
  };
};

const acceptCall = () => {
  const acceptParams = {
    callOption: {
      localMediaView: document.getElementById("caller") as HTMLMediaElement,
      remoteMediaView: document.getElementById("callee") as HTMLMediaElement,
      audioEnabled: true,
      videoEnabled: true,
    },
  };

  currentCall.value?.accept(acceptParams);
  currentState.value = "IN_CALL";
};

const declineCall = () => {
  currentState.value = "IDLE";
  currentCall.value = null;
};

const endCall = () => {
  currentCall.value?.end();
  currentState.value = "IDLE";
  currentCall.value = null;
};
</script>
