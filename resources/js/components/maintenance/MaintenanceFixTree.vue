<template>
	<Card v-if="data !== undefined && fixable" class="min-h-40 dark:bg-surface-800 shadow shadow-surface-950/30 rounded-lg relative">
		<template #title>
			<div class="text-center">
				{{ $t("maintenance.fix-tree.title") }}
			</div>
		</template>
		<template #content>
			<ScrollPanel class="w-full h-40 text-sm text-muted-color">
				<div class="w-full text-left" v-if="!loading">
					{{ $t("maintenance.fix-tree.Oddness") }}: {{ data.oddness }}<br />
					{{ $t("maintenance.fix-tree.Duplicates") }}: {{ data.duplicates }}<br />
					{{ $t("maintenance.fix-tree.Wrong parents") }}: {{ data.wrong_parent }}<br />
					{{ $t("maintenance.fix-tree.Missing parents") }}: {{ data.missing_parent }}<br />
				</div>
				<ProgressSpinner v-if="loading" class="w-full"></ProgressSpinner>
			</ScrollPanel>
			<!-- <div class="flex gap-4 mt-1">
				<Button v-if="fixable && !loading" severity="primary" class="w-full border-none" @click="exec">{{
					$t("maintenance.fix-tree.button")
				}}</Button>
			</div> -->
		</template>
	</Card>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
// import Button from "primevue/button";
import Card from "primevue/card";
import { useToast } from "primevue/usetoast";
import ProgressSpinner from "primevue/progressspinner";
import ScrollPanel from "primevue/scrollpanel";
import MaintenanceService from "@/services/maintenance-service";

const data = ref<App.Http.Resources.Diagnostics.TreeState | undefined>(undefined);
const loading = ref(false);
const toast = useToast();

const fixable = computed(() => {
	return data.value && (data.value.oddness > 0 || data.value.duplicates > 0 || data.value.wrong_parent > 0 || data.value.missing_parent > 0);
});
function load() {
	MaintenanceService.treeGet().then((response) => {
		data.value = response.data;
	});
}

// function exec() {
// 	loading.value = true;
// 	MaintenanceService.treeDo().then((response) => {
// 		toast.add({ severity: "success", summary: "Success", life: 3000 });
// 		loading.value = false;
// 	}).catch((e) => {
// 		toast.add({ severity: "error", summary: "Error", detail: e.response.data.message,  life: 3000 });
// 		loading.value = false;
// 	});
// }

load();
</script>

<style lang="css" scoped>
.lychee-dark .p-card {
	--p-card-background: var(--p-surface-800);
}
</style>
