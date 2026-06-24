<script setup>
import { ref } from 'vue';

const pdsBaseDomain = ref('');
const actor = ref('');
const password = ref('');
const mfaToken = ref('')
const deleteToken = ref('');

const didToDelete = ref('');

async function requestDelete() {
  if (!pdsBaseDomain.value || !actor.value || !password.value) {
    window.alert("Please fill in your PDS domain, email/handle/DID, and password.");
    return;
  }

  const sessionStartData = {
    identifier: actor.value,
    password: password.value,
    allowTakendown: true,
    ...(mfaToken.value ? { authFactorToken: mfaToken.value.trim() } : {})
  }

  const sessionResponse = await fetch(
    `https://${pdsBaseDomain.value}/xrpc/com.atproto.server.createSession`,
    {
      method: "POST",
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(sessionStartData)
    }
  );

  if (sessionResponse.status === 400) {
    const sessionResponseData = await sessionResponse.json();
    if (sessionResponseData.error === "AuthFactorTokenRequired") {
      window.alert("Check your email for an MFA prompt and enter the token in the MFA Token box.");
    }
    else {
      window.alert("Something went wrong (response code 400).");
    }
  }
  else if (sessionResponse.status === 200) {
    const sessionResponseData = await sessionResponse.json();
    didToDelete.value = sessionResponseData.did;
    const deleteRequestResponse = await fetch(
      `https://${pdsBaseDomain.value}/xrpc/com.atproto.server.requestAccountDelete`,
      {
        method: "POST",
        headers: {
          'Authorization': `Bearer ${sessionResponseData.accessJwt}`
        }
      }
    );

    if (deleteRequestResponse.status === 200) {
      window.alert("Check your email for the deletion token. Move to Step 2.")
    } else {
      window.alert("Something went wrong (could not request deletion token).")
    }
  } else {
    window.alert("Something went wrong (could not start session).")
  }
}

async function finalizeDeletion() {
    if (!didToDelete) {
      window.alert("You didn't request deletion first!");
      return
    }
    if (!deleteToken.value) {
      window.alert("Please enter the deletion token from your email.");
      return;
    }

    const finalizeDeleteData = {
      'did': didToDelete.value,
      'password': password.value,
      'token': deleteToken.value.trim()
    }

    const finalizeDeleteResponse = await fetch(
      `https://${pdsBaseDomain.value}/xrpc/com.atproto.server.deleteAccount`,
      {
        method: "POST",
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(finalizeDeleteData)
      }
    );

    if (finalizeDeleteResponse.status === 200) {
      window.alert("Account deleted.")
    } else {
      window.alert("Something went wrong (could not delete account).")
    }
  }
</script>

<template>
  <div>
    <NuxtRouteAnnouncer />
    <h1>SkyWipe</h1>
    <p>
      Use this page if you've recently done a PDS migration and need to delete your old account. <br>
      You'll need your email, handle, or DID, in addition to the password for your old PDS account.
    </p>
    <div id="requestzone">
      <h2>Step 1: Request Delete Token</h2>
      <p>
        Enter your old PDS domain, email/handle/DID, and password; then click "Request Deletion". <br>
        You'll get an email from your old PDS provider (Bluesky or other). <br>
        MFA enabled? Start by entering just your email/handle/DID and password, then click the button.
      </p>
      <div class="inputgroup">
        <input v-model="pdsBaseDomain" type="text" placeholder="Old PDS domain (bsky.social, etc)">
        <input v-model="actor" type="text" placeholder="Email/Handle/DID">
        <input v-model="password" type="password" placeholder="Password (not App Password)">
        <input v-model="mfaToken" type="text" placeholder="MFA Token (XXXXX-XXXXX)">
      </div>
      <button @click="requestDelete">Request Deletion</button>
    </div>
    <div id="deletezone">
      <h2>Step 2: Finalize Deletion</h2>
      <p>
        Enter the deletion token you got in your email. <br>
        This will finalize the deletion.
      </p>
      <p v-if="didToDelete">
        You're deleting {{didToDelete}}.
      </p>
      <input id="dTokenInput" v-model="deleteToken" type="text" placeholder="XXXXX-XXXXX">
      <button @click="finalizeDeletion">Finalize Deletion</button>
    </div>
  </div>
</template>
