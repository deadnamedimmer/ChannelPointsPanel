<script lang="ts">
  import { onMount } from "svelte";

  let access_token = "";
  let auth_bearer = "";
  let channelPointsOn;
  let channelRewards = [
        {'title': 'Good Job!',
        'cost': 500,
        'background_color': '#392e5c'},
        {'title': 'ba dum tss',
        'cost': 1000,
        'background_color': '#392e5c'},
        {'title': 'OOF',
        'cost': 1000,
        'background_color': '#392e5c'},
        {'title': 'Hello There!',
        'cost': 1000,
        'background_color': '#392e5c'},
        {'title': 'Laugh Track',
        'cost': 1000,
        'background_color': '#392e5c'},
        {'title': 'gta-v-death',
        'cost': 1000,
        'background_color': '#392e5c'},
        {'title': 'bonk',
        'cost': 1000,
        'background_color': '#392e5c'},
        {'title': 'Bring out the GOOSE',
        'cost': 2500,
        'background_color': '#392e5c'},
        {'title': 'realistic knocking',
        'cost': 3000,
        'background_color': '#392e5c'},
        {'title': 'Metal Pipe',
        'cost': 5000,
        'background_color': '#392e5c'},
        {'title': 'RickRoll',
        'cost': 25000,
        'background_color': '#E69900',
        'is_max_per_stream_enabled': true,
        'max_per_stream': 1},
    ];

  //Get our Twitch Credentials, wait to render front end until credentials, run onMount
  async function getTokens(): Promise<string> {
    let requestData = {
        client_id: import.meta.env.VITE_CLIENT_ID,
        client_secret: import.meta.env.VITE_CLIENT_SECRET,
        grant_type: 'refresh_token',
        refresh_token: import.meta.env.VITE_REFRESH_TOKEN,
    };

    let formBody;
    formBody = [];
    for (var property in requestData) {
        let encodedKey = encodeURIComponent(property);
        let encodedValue = encodeURIComponent(requestData[property]);
        formBody.push(encodedKey + "=" + encodedValue);
    }
    formBody = formBody.join("&");

    let response = await fetch('https://id.twitch.tv/oauth2/token', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: formBody,
    })

    let data = await response.json()
    return data.access_token;
  }

  //Create our channel point rewards
  async function createAllChannelRewards() {
    channelRewards.forEach((reward) => {
        createSingleChannelReward(auth_bearer, reward)
    })
  }

  async function createSingleChannelReward(auth_bearer, reward) {
    let url = "https://api.twitch.tv/helix/channel_points/custom_rewards?broadcaster_id=79176881"
    let createChannelReward = new Request(
        url,
        {
            method: 'POST',
            headers: {
                "Client-Id": import.meta.env.VITE_CLIENT_ID, 
                "Authorization": auth_bearer,
                "Content-Type": "application/json",
            },
            body: JSON.stringify(reward)
        }
    )
    let response = await fetch(createChannelReward);
    let data = await response.json();
    console.log("Created " + reward.title)
  }

  //Get our list of Channel Point Rewards
  async function getRewards() {
    const getRewardsRequest = new Request(
        "https://api.twitch.tv/helix/channel_points/custom_rewards?broadcaster_id=79176881", 
        {
            method: 'GET', 
            headers: {
                "Client-Id": import.meta.env.VITE_CLIENT_ID, 
                "Authorization": auth_bearer,
            },
        }
    );

    let response = await fetch(getRewardsRequest)
    let data = await response.json()
    return data
  }
  
  //Get the current status of the rewards
  async function getRewardStatus() {
    let rewardsList = await getRewards();
    let activeRewards = 0;

    await rewardsList.data.forEach((value) => {
      if (value.is_enabled) {
        activeRewards++
      }
    })

    if (activeRewards/rewardsList.data.length > .75) {
      return true
    } else {
      return false
    }
  }

  async function updateReward(rewardID, shouldBeActive) {
        let url = "https://api.twitch.tv/helix/channel_points/custom_rewards?broadcaster_id=79176881&id=" + rewardID;
        let updateRewardsRequest = new Request(
            url,
            {
                method: 'PATCH',
                headers: {
                    "Client-Id": import.meta.env.VITE_CLIENT_ID, 
                    "Authorization": auth_bearer,
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                    'is_enabled': shouldBeActive
                })
            }
        )
        let response = await fetch(updateRewardsRequest)
        let data = await response.json()
        return true
    }
  
  async function updateAllRewards(shouldBeActive) {
  let rewards = await getRewards();
  let updatePromises = rewards.data.map((reward) => {
      return updateReward(reward.id, shouldBeActive);
  });
  await Promise.all(updatePromises);
  getRewardStatus().then((value) => {
    channelPointsOn = value;
  })
}

  //Edit our channel point rewards
  
  async function turnOn() {
    updateAllRewards(true)
  }

  async function turnOff() {
    updateAllRewards(false)
  }

  onMount(() => {
    getTokens().then((value) => {
      access_token = value;
      console.log("Loaded Tokens")
      auth_bearer = "Bearer " + access_token

      getRewardStatus().then((value) => {
        channelPointsOn = value;
      })
    })
  })
</script>

<main>
  <h2>Channel Points are</h2>
  {#if access_token !== ""}
    <!-- <button on:click={createAllChannelRewards}>Create Channel Rewards</button> -->
    <button on:click={turnOn} style="background-color: {channelPointsOn == true ? "purple" : "grey"};">On</button>
    <button on:click={turnOff} style="background-color: {channelPointsOn == false ? "purple" : "grey"};">Off</button>
  {/if}
</main>

<style>
</style>