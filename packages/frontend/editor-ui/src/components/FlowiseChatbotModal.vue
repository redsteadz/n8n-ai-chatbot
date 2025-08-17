<script setup lang="ts">
import { onMounted, onBeforeUnmount } from 'vue';
import type { EventBus } from '@n8n/utils/event-bus';
import { createEventBus } from '@n8n/utils/event-bus';
import Modal from './Modal.vue';
import { FLOWISE_CHATBOT_MODAL_KEY } from '@/constants';
import { useI18n } from '@n8n/i18n';

const props = withDefaults(
	defineProps<{
		modalName: string;
		modalBus?: EventBus;
		data?: Record<string, unknown>;
	}>(),
	{
		modalBus: () => createEventBus(),
		data: () => ({}),
	},
);

const i18n = useI18n();
const modalBus = props.modalBus;

let flowiseScript: HTMLScriptElement | null = null;

const initializeFlowise = () => {
	// Load the Flowise script
	flowiseScript = document.createElement('script');
	flowiseScript.type = 'module';
	flowiseScript.innerHTML = `
		import Chatbot from "https://cdn.jsdelivr.net/npm/flowise-embed/dist/web.js"
		Chatbot.init({
			chatflowid: "80cbcf69-627f-4667-9719-ac4c5e224e36",
			apiHost: "http://localhost:3000",
		})
	`;
	document.head.appendChild(flowiseScript);
};

const cleanupFlowise = () => {
	// Clean up the script when modal is closed
	if (flowiseScript) {
		document.head.removeChild(flowiseScript);
		flowiseScript = null;
	}

	// Remove any Flowise DOM elements
	const flowiseElements = document.querySelectorAll('[id*="flowise"], [class*="flowise"]');
	flowiseElements.forEach((element) => element.remove());
};

const closeDialog = () => {
	cleanupFlowise();
	modalBus.emit('close');
};

onMounted(() => {
	initializeFlowise();
});

onBeforeUnmount(() => {
	cleanupFlowise();
});
</script>

<template>
	<Modal
		width="800px"
		height="600px"
		:title="i18n.baseText('flowiseChatbot.title')"
		:event-bus="modalBus"
		:name="modalName"
		:center="true"
	>
		<template #content>
			<div :class="$style.container">
				<div :class="$style.chatContainer">
					<!-- The Flowise chatbot will be injected here automatically -->
					<div id="flowise-chatbot-container"></div>
				</div>
			</div>
		</template>

		<template #footer>
			<div class="action-buttons">
				<n8n-button float="right" :label="i18n.baseText('generic.close')" @click="closeDialog" />
			</div>
		</template>
	</Modal>
</template>

<style module lang="scss">
.container {
	display: flex;
	flex-direction: column;
	height: 100%;
	min-height: 500px;
}

.chatContainer {
	flex: 1;
	width: 100%;
	height: 100%;
	overflow: hidden;

	// Ensure the Flowise chatbot fills the container
	:deep(#flowise-chatbot) {
		width: 100% !important;
		height: 100% !important;
		border: none !important;
	}
}
</style>
