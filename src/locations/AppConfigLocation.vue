<template>
  <h2>Place Demo</h2>
  <div class="demo-container">
    <div class="place" v-for="(place, index) in places">
      <div class="place-info">
        <p>{{ place.fields.name[locale] }}</p>
        <p></p>
      </div>

      <img
        class="place-image"
        :src="place.image.file[locale].url"
        :alt="place.image.description[locale]"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import type { ConfigAppSDK } from "@contentful/app-sdk";

// eslint-disable-next-line @typescript-eslint/no-empty-interface
export interface AppInstallationParameters {}

const props = defineProps<{ sdk: ConfigAppSDK }>();

let parameters = {};
const places = ref([]);
const locale = "en-US";

const onConfigure = async () => {
  // This method will be called when a user clicks on "Install"
  // or "Save" in the configuration screen.
  // for more details see https://www.contentful.com/developers/docs/extensibility/ui-extensions/sdk-reference/#register-an-app-configuration-hook

  // Get current the state of EditorInterface and other entities
  // related to this app installation
  const currentState = await props.sdk.app.getCurrentState();

  return {
    // Parameters to be persisted as the app configuration.
    parameters,
    // In case you don't want to submit any update to app
    // locations, you can just pass the currentState as is
    targetState: currentState,
  };
};

props.sdk.app.onConfigure(() => onConfigure());

const getImageUrl = async (id: string) => {
  try {
    const imageAsset = await props.sdk.space.getAsset(id);
    return imageAsset.fields; // Return the asset's URL
  } catch (error) {
    console.error(`Error fetching image asset with ID ${id}:`, error);
    return null; // Return null in case of an error
  }
};

onMounted(async () => {
  const params: AppInstallationParameters | null =
    await props.sdk.app.getParameters();
  parameters = params || parameters;

  const contentsOfType = await props.sdk.space.getEntries({
    content_type: "place",
  });

  // Map through entries and resolve image URLs
  const mappedPlaces = await Promise.all(
    contentsOfType.items.map(async (entry) => {
      const imageId = entry.fields.place_image?.[locale]?.sys?.id;
      const image = imageId ? await getImageUrl(imageId) : null;

      const entryObj = {
        fields: entry.fields,
        image,
      };

      console.log("Liam:entryObj");
      console.log(entryObj);

      return entryObj;
    })
  );

  console.log("Liam:mappedPlaces");
  console.log(mappedPlaces);

  places.value = mappedPlaces; // Assign the resolved places to reactive variable

  await props.sdk.app.setReady();
});
</script>

<style>
body {
  background-color: #f0f0f0;
}

.demo-container {
  display: flex;
  flex-direction: column;
  margin: 2rem;
}

.place {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  margin-bottom: 1rem;
}
</style>
