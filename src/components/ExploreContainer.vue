<template>
  <div
    id="container"
    style="
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
    "
  >
    <div v-if="!selected">
      <p>Choose player</p>
      <div>
        <ion-button @click="() => (selected = 'me')">Me</ion-button>
        <ion-button @click="() => (selected = 'friend')">Friend</ion-button>
      </div>
    </div>

    <div v-if="selected">
      <SendbirdContext :player="selected" />
    </div>
    <!-- <strong>You are {{ !isMe ? "Me" : "Friend" }}</strong>
    <ion-button
      :disabled="loading"
      :expand="loading ? 'block' : 'full'"
      @click="setIsMeState(false)"
      >Be Me</ion-button
    >
    <ion-button @click="setIsMeState(true)">Be Friend</ion-button>
    <ion-button @click="connect(isMe)">Connect</ion-button>
    <div style="display: flex; gap: 2rem; margin-block: 2rem">
      <video
        id="caller"
        autoplay
        style="width: 100px; height: 100px; border: 1px solid dodgerblue"
      ></video>
      <video
        id="callee"
        autoplay
        style="width: 100px; height: 100px; border: 1px solid dodgerblue"
      ></video>
    </div>

    <ion-button @click="doCall(isMe)">Call other</ion-button>
    <ion-button>Answer call</ion-button>
    <ion-button>Hang up</ion-button> -->
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import SendBirdCall, { DialParams, AcceptParams } from "sendbird-calls";
import { IonButton } from "@ionic/vue";
import { Plugins } from "@capacitor/core";
import SendbirdContext from "./SendbirdContext.vue";
const { Permissions } = Plugins;
const connect = async (isMe: boolean) => {
  // SendBirdCall.useMedia({ video: true, audio: true });
  const authOption = {
    userId: isMe ? "p3aceful" : "friend",
    accessToken: isMe
      ? "6db55614400c54b26249687c00252ddbff505a68"
      : "7a030c1b57be84d071258606c9a3df5f40c6048c",
  };

  await SendBirdCall.authenticate(authOption, (result, error) => {
    if (error) {
      // Handle authentication failure.
      console.log("Auth error", { error });
    } else {
      // The user has been successfully authenticated and is connected to Sendbird server.
      console.log("Successfully authenticated", { result });
    }
  });

  await SendBirdCall.connectWebSocket()
    .then((res) => {
      console.log("Websocket connection success", { res });
    })
    .catch((err) => console.log("Websocket connection Bummer!", { err }));

  SendBirdCall.addListener("HANDLER", {
    onRinging: (call) => {
      // call.onEstablished = (call) => {};

      // call.onConnected = (call) => {};

      // call.onEnded = (call) => {};

      // call.onRemoteAudioSettingsChanged = (call) => {};

      // call.onRemoteVideoSettingsChanged = (call) => {};

      const acceptParams: AcceptParams = {
        callOption: {
          localMediaView: document.getElementById(
            "local_video_element_id"
          ) as HTMLMediaElement,
          remoteMediaView: document.getElementById(
            "remote_video_element_id"
          ) as HTMLMediaElement,
          audioEnabled: true,
          videoEnabled: true,
        },
      };

      call.accept(acceptParams);
    },
  });
  SendBirdCall.useMedia({ video: true, audio: true });
};

const doCall = (isMe: boolean) => {
  const dialParams: DialParams = {
    userId: isMe ? "friend" : "p3aceful",
    isVideoCall: true,
    callOption: {
      localMediaView: document.querySelector("#caller") as HTMLMediaElement,
      remoteMediaView: document.querySelector("#callee") as HTMLMediaElement,
      audioEnabled: true,
      videoEnabled: true,
    },
  };

  const call = SendBirdCall.dial(dialParams, (call, error) => {
    if (error) {
      // Dialing failed.
      console.log("Dialing failed!");
    }
  });
};

export default defineComponent({
  name: "ExploreContainer",
  components: {
    IonButton,
    SendbirdContext,
  },
  props: {
    name: String,
  },
  setup() {
    console.log("Test");
    const loading = ref(false);
    const isMe = ref(false);
    const setIsMeState = (value: boolean) => (isMe.value = value);
    const onUserClick = async (user: any) => {
      loading.value = true;
      await user();
      loading.value = false;
    };

    const selected = ref<string | null>(null);
    return {
      selected,
      onUserClick,
      loading,
      isMe,
      setIsMeState,
      doCall,
      connect,
    };
  },
});
</script>

<style scoped>
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>
