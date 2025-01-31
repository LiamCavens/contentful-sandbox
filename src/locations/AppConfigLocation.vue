<template>
  <h2>Place Demo</h2>
  <div class="demo-container">
    <div class="place" v-for="(place, index) in places">
      <div class="place-info">
        <div class="place-info-main">
          <div class="place-info-text">
            <p>{{ place.fields.name[locale] }}</p>
            <Vueform>
              <SelectElement
                name="select"
                :placeholder="'eating'"
                :items="['Vue.js', 'React', 'AngularJS']"
              />
            </Vueform>
            <Vueform>
              <SelectElement
                name="select"
                :placeholder="'status'"
                :items="['Vue.js', 'React', 'AngularJS']"
              />
            </Vueform>
            <p>£25</p>
            <p>8:00 - 20:00</p>

            <!-- above should be an anchor tag -->
            <a v-if="place.fields.website" :href="place.fields.website[locale]">{{ place.fields.name[locale] }} Website</a>
          </div>
          <div class="place-info-map">MAP PLACEHOLDER</div>
        </div>
        <div v-if="place.fields.description" class="place-info-notes">
          <p>{{ place.fields.description[locale] }}</p>
        </div>
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
  background-color: #fff;
  padding: 1rem;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  margin-bottom: 1rem;
}

.place-info,
.place-info-text {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  gap: 1rem;
}

.place-info {
  flex: 1;
}

.place-info-text {
  width: 100%;
  padding-right: 5rem;
}

.place-info-map {
  width: 100%;
  border: 1px solid black;
  padding: 1rem;
}

.place-info-text p {
  margin: 0;
  padding: 0;
}

.place-info-main {
  padding-right: 1rem;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

.place-info-notes {
  background-color: #f0f0f0;
  border: 1px solid #ccc;
  padding: 1rem;
  margin-right: 1rem;
}
</style>
