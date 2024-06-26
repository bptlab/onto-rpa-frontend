<template>
  <div class="h-screen max-h-screen flex flex-col">
    <div
      class="bg-sky-700 text-center text-slate-100 py-12 flex-initial flex justify-around"
    >
      <div class="flex-1">
        <router-link :to="{ name: 'Overview' }" title="Back to Overview">
          <FontAwesomeIcon
            class="cursor-pointer"
            :icon="faChevronLeft"
            size="2xl"
          />
        </router-link>
      </div>

      <div
        class="text-white bg-transparent text-4xl border-0 border-b-2 w-4/5 shadow-none"
      >
        {{ botModel.name }}
      </div>

      <div class="flex-1">
        <FontAwesomeIcon
          class="ml-4 cursor-pointer"
          :icon="faCamera"
          size="2xl"
          @click="takeScreenshot"
        />
      </div>
    </div>
    <div class="flex-auto grid grid-cols-10">
      <MetricsSidebar
        class="col-span-2 drop-shadow-lg bg-white"
        :modelMetrics="modelMetrics"
      ></MetricsSidebar>
      <BotModelerCanvas
        v-if="botModel.model"
        :diagram="botModel.model"
        :key="modelerKey"
        @modeler-shown="modelerLoaded"
        @modeler-selection-changed="selectionChanged"
        @modeler-element-changed="elementChanged"
        class="col-span-8"
      ></BotModelerCanvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import BotModelerCanvas from "../components/BotModeler/BotModelerCanvas.vue";
import {
  ModelerElement,
  ModelerEvent,
  ModelerSelectionChange,
} from "../interfaces/ModelerEvents";
import BotModel from "../interfaces/BotModel";
import botModelApi from "../api/botModelApi";
import { useToast } from "vue-toastification";
import { useRouter, useRoute } from "vue-router";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { faChevronLeft } from "@fortawesome/free-solid-svg-icons";
import { faCamera } from "@fortawesome/free-solid-svg-icons";
import { BotModelMetrics } from "../interfaces/BotModelMetrics";
import MetricsSidebar from "../components/BotModelMetrics/ModelMetricsSidebar.vue";
import BotModelMetricsCalculator from "../utils/BotModelMetricsCalculator";

const modelerShown = ref(false);
const modeler = ref({} as any);
const selectedElements = ref([] as ModelerElement[]);
const element = ref({} as ModelerElement | null);
const botModel = ref({} as BotModel);
const modelerKey = ref(0);
let modelMetricsCalculator = {} as BotModelMetricsCalculator;

const toast = useToast();
const router = useRouter();
const route = useRoute();

const modelMetrics = ref({} as BotModelMetrics);

onMounted(async () => {
  if (!route.params.modelId) {
    toast.error("Invalid URL, you must provide a bot model id.");
    router.push({ name: "Overview" });
  }
  try {
    botModel.value = await botModelApi.getBotModel(
      route.params.modelId as string
    );
  } catch (e) {
    toast.error("Could not load the requested bot model.");
    router.push({ name: "Overview" });
  }
  modelMetricsCalculator = new BotModelMetricsCalculator(
    botModel.value.processTree
  );
  updateMetrics();
});

function modelerLoaded(loadedModeler): void {
  modeler.value = loadedModeler;
  modelerShown.value = true;
}

function takeScreenshot() {
  toast.warning("Not yet supported.");
}

function updateMetrics() {
  modelMetrics.value = modelMetricsCalculator.getModelMetrics();
}

function selectionChanged(e: ModelerSelectionChange) {
  selectedElements.value = e.newSelection;
  if (e.newSelection.length > 0) {
    element.value = e.newSelection[0];
  } else {
    element.value = null;
  }
}

function elementChanged(e: ModelerEvent) {
  // this.saveDiagram();
  if (!element.value || !element.value.businessObject) {
    return;
  }
  if (element.value.businessObject.id === e.element.businessObject.id) {
    element.value = e.element;
  }
}
</script>

<style>
.djs-palette {
  display: none;
}
</style>
