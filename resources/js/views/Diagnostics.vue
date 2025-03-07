<template>
	<Toolbar class="w-full border-0 h-14">
		<template #start>
			<Button @click="togglableStore.toggleLeftMenu" icon="pi pi-bars" class="mr-2 border-none" severity="secondary" text />
		</template>

		<template #center>
			{{ $t("lychee.DIAGNOSTICS") }}
		</template>

		<template #end>
			<Button :disabled="!canCopy" text aria-label="Copy" icon="pi pi-copy" v-tooltip="'Copy diagnostics to clipboard'" @click="copy" />
		</template>
	</Toolbar>
	<ErrorsDiagnotics @loaded="loadError" />
	<InfoDiagnostics v-if="user?.id" @loaded="loadInfo" />
	<SpaceDiagnostics v-if="user?.id" />
	<ConfigurationsDiagnostics v-if="user?.id" @loaded="loadConfig" />
</template>
<script setup lang="ts">
import { ref, computed } from "vue";
import Toolbar from "primevue/toolbar";
import Button from "primevue/button";
import ConfigurationsDiagnostics from "@/components/diagnostics/ConfigurationsDiagnostics.vue";
import InfoDiagnostics from "@/components/diagnostics/InfoDiagnostics.vue";
import ErrorsDiagnotics from "@/components/diagnostics/ErrorsDiagnostics.vue";
import SpaceDiagnostics from "@/components/diagnostics/SpaceDiagnostics.vue";
import { useAuthStore } from "@/stores/Auth";
import { storeToRefs } from "pinia";
import { useTogglablesStateStore } from "@/stores/ModalsState";
import { useToast } from "primevue/usetoast";

const authStore = useAuthStore();
const { user } = storeToRefs(authStore);
authStore.getUser();
const togglableStore = useTogglablesStateStore();

const toast = useToast();

const errorLoaded = ref<App.Http.Resources.Diagnostics.ErrorLine[] | undefined>(undefined);
const infoLoaded = ref<string[] | undefined>(undefined);
const configurationLoaded = ref<string[] | undefined>(undefined);

const canCopy = computed(() => errorLoaded.value !== undefined && infoLoaded.value !== undefined && configurationLoaded.value !== undefined);

function loadError(val: App.Http.Resources.Diagnostics.ErrorLine[]) {
	errorLoaded.value = val;
}

function loadInfo(val: string[]) {
	infoLoaded.value = val;
}

function loadConfig(val: string[]) {
	configurationLoaded.value = val;
}

function copy() {
	if (!canCopy) {
		return;
	}

	const errorText = errorLoaded.value
		?.filter((errorLine) => errorLine.type !== undefined)
		.map((errorLines) => `${errorLines.type.padEnd(7)}: ${errorLines.line}`)
		.join("\n");
	const infoText = infoLoaded.value?.join("\n");
	const configurationText = configurationLoaded.value?.join("\n");

	const toClipBoard = `Errors:\n${"-".repeat(20)}\n${errorText}\n\n\nInfo:\n${"-".repeat(20)}\n${infoText}\n\n\nConfig:\n${"-".repeat(20)}\n${configurationText}`;

	navigator.clipboard
		.writeText(toClipBoard)
		.then(() => toast.add({ severity: "info", summary: "Info", detail: "Diagnostic copied to clipboard", life: 3000 }));
}
</script>
