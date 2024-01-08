<script setup lang="ts">
import { createClient } from "@supabase/supabase-js";
import { Ref, computed, ref } from "vue";

type Theme = {
  activated?: string;
  culled: boolean;
  title: string;
};

const PUBLIC_API_KEY =
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im9mcGVyd2pybHVycGtncXRzcnVlIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MDQ0NzExMTgsImV4cCI6MjAyMDA0NzExOH0.KipFHbAuVdRRZr4rieMXsDvAyyUBUZiLuz-uEf95SKg";
const supabase = createClient(
  "https://ofperwjrlurpkgqtsrue.supabase.co",
  PUBLIC_API_KEY
);

const themes: Ref<Array<Theme>> = ref([]);

const loadThemes = async () => {
  const { data, error } = await supabase.from("theme").select().order("title");

  if (data) {
    themes.value = data;
  }

  if (error) {
    console.error(error);
  }
};

loadThemes();

supabase
  .channel("table-db-changes")
  .on(
    "postgres_changes",
    {
      event: "*",
      schema: "public",
      table: "theme",
    },
    () => loadThemes()
  )
  .subscribe();

enum Tab {
  Culled = "Culled",
  Previous = "Previous",
  Upcoming = "Upcoming",
}

const currentTab = ref(Tab.Upcoming);

const displayedThemes = computed(() => {
  switch (currentTab.value) {
    case Tab.Culled:
      return themes.value.filter(({ culled }) => culled);
    case Tab.Previous:
      const previousThemes = themes.value.filter(({ activated }) => activated);

      previousThemes?.sort((a, b) => {
        if (!a.activated || !b.activated) {
          return 0;
        }

        if (a.activated < b.activated) {
          return -1;
        }
        if (a.activated > b.activated) {
          return 1;
        }

        return 0;
      });

      return previousThemes;
    case Tab.Upcoming:
      return themes.value.filter(
        ({ activated, culled }) => !activated && !culled
      );
  }
});
</script>

<template>
  <div>
    <h1 class="pt-8 text-4xl font-thin text-center">Theme UI</h1>

    <div
      v-if="themes"
      class="flex flex-col items-center mx-4 mt-8 shadow-xl md:mx-32 lg:mx-64"
    >
      <div class="flex w-full text-2xl">
        <div
          v-for="tab in Object.values(Tab)"
          class="px-4 text-center rounded-t-lg transition-all cursor-pointer grow hover:bg-gray-500"
          :class="`${tab == currentTab ? 'bg-gray-600' : 'bg-gray-800'}`"
          @click="currentTab = tab"
        >
          <span class="font-thin">{{ tab }}</span>
        </div>
      </div>
      <div class="flex flex-col w-full">
        <div
          v-for="({ activated, title }, index) in displayedThemes"
          :data-index="index"
          class="flex justify-between px-4 py-1 w-full font-thin theme"
        >
          <span>
            {{ title }}
          </span>
          <span v-if="currentTab == Tab.Previous">
            {{ activated }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="postcss">
.card {
  @apply bg-white rounded-md;
}

.theme:nth-child(odd) {
  @apply bg-gray-700 shadow-inner;
}

.theme:nth-child(even) {
  @apply bg-gray-600;
}

.theme:last-child {
  @apply rounded-b-lg;
}
</style>
