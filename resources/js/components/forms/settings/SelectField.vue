<template>
	<div class="flex gap-4 items-center justify-between">
		<label :for="props.config.key" class="w-full" :class="props.config.require_se ? 'text-primary-emphasis' : 'text-muted-color-emphasis'">{{
			props.config.documentation
		}}</label>
		<div class="flex gap-4 items-center">
			<ResetField v-if="changed" @click="reset" />
			<Select :id="props.config.key" class="border-none" v-model="val" :options="options" @update:modelValue="update" />
		</div>
	</div>
</template>

<script setup lang="ts" generic="T extends string">
import { computed, ref, watch } from "vue";
import Select from "primevue/select";
import ResetField from "@/components/forms/settings/ResetField.vue";

type Props = {
	config: App.Http.Resources.Models.ConfigResource;
};

const props = defineProps<Props>();

const val = ref(props.config.value);
const options = ref(props.config.type.split("|"));

const changed = computed(() => val.value !== props.config.value);

const emits = defineEmits<{
	filled: [key: string, value: string];
	reset: [key: string];
}>();

function update() {
	emits("filled", props.config.key, val.value);
}

function reset() {
	emits("reset", props.config.key);
	val.value = props.config.value;
}

// We watch props in case of updates.
watch(
	() => props.config,
	(newValue, _oldValue) => (val.value = newValue.value),
);
</script>
