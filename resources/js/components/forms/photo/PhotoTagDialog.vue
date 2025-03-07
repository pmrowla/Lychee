<template>
	<Dialog v-model:visible="visible" pt:root:class="border-none" modal :dismissable-mask="true">
		<template #container="{ closeCallback }">
			<div class="p-9 text-center text-muted-color">
				<p class="mb-5 text-sm/4">
					{{ question }}
				</p>
				<div class="my-3 first:mt-0 last:mb-0">
					<AutoComplete
						id="tags"
						v-model="tags"
						:typeahead="false"
						multiple
						class="pt-3 border-b hover:border-b-0"
						:placeholder="$t('lychee.NO_TAGS')"
						pt:inputmultiple:class="w-full border-t-0 border-l-0 border-r-0 border-b hover:border-b-primary-400 focus:border-b-primary-400"
					/>
				</div>
				<div>
					<Checkbox v-model="shallOverride" :binary="true" inputId="shallOverride" />
					<label for="shallOverride" class="ml-2 text-sm text-muted-color">{{ $t("lychee.TAGS_OVERRIDE_INFO") }}</label>
				</div>
			</div>
			<div class="flex">
				<Button severity="secondary" class="font-bold w-full border-none rounded-none rounded-bl-xl" @click="close">
					{{ $t("lychee.CANCEL") }}
				</Button>
				<Button severity="contrast" class="font-bold w-full border-none rounded-none rounded-br-xl" @click="execute">
					{{ $t("lychee.PHOTO_SET_TAGS") }}
				</Button>
			</div>
		</template>
	</Dialog>
</template>
<script setup lang="ts">
import { computed, ref } from "vue";
import PhotoService from "@/services/photo-service";
import AlbumService from "@/services/album-service";
import { sprintf } from "sprintf-js";
import { useToast } from "primevue/usetoast";
import Button from "primevue/button";
import Dialog from "primevue/dialog";
import { trans } from "laravel-vue-i18n";
import Checkbox from "primevue/checkbox";
import AutoComplete from "primevue/autocomplete";

const props = defineProps<{
	parentId: string | undefined;
	photo?: App.Http.Resources.Models.PhotoResource;
	photoIds?: string[];
}>();

const visible = defineModel<boolean>("visible", { default: false });

const emits = defineEmits<{
	tagged: [];
}>();

const toast = useToast();

const question = computed(() => {
	if (props.photo) {
		return trans("lychee.PHOTO_NEW_TAGS");
	}
	return sprintf(trans("lychee.PHOTOS_NEW_TAGS"), props.photoIds?.length);
});

const shallOverride = ref(false);
const tags = ref<string[]>([]);

function close() {
	visible.value = false;
	tags.value = [];
	shallOverride.value = false;
}

function execute() {
	if (tags.value === undefined) {
		return;
	}

	let photoTaggedIds = [];
	if (props.photo) {
		photoTaggedIds.push(props.photo.id);
	} else {
		photoTaggedIds = props.photoIds as string[];
	}

	PhotoService.tags(photoTaggedIds, tags.value, shallOverride.value).then(() => {
		toast.add({
			severity: "success",
			summary: "tags updated",
			life: 3000,
		});
		AlbumService.clearCache(props.parentId);
		close();
		emits("tagged");
	});
}
</script>
