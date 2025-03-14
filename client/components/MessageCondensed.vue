<template>
	<div :class="['msg', {closed: isCollapsed}]" data-type="condensed">
		<div class="condensed-summary">
			<span class="time" />
			<span class="from" />
			<span class="content" @click="onCollapseClick"
				>{{ condensedText
				}}<button class="toggle-button" aria-label="Toggle status messages"
			/></span>
		</div>
		<Message
			v-for="message in messages"
			:key="message.id"
			:network="network"
			:message="message"
		/>
	</div>
</template>

<script>
const constants = require("../js/constants");
import Message from "./Message.vue";

export default {
	name: "MessageCondensed",
	components: {
		Message,
	},
	props: {
		network: Object,
		messages: Array,
		keepScrollPosition: Function,
		focused: Boolean,
	},
	data() {
		return {
			isCollapsed: true,
		};
	},
	computed: {
		condensedText() {
			const obj = {};

			constants.condensedTypes.forEach((type) => {
				obj[type] = 0;
			});

			for (const message of this.messages) {
				// special case since one MODE message can change multiple modes
				if (message.type === "mode") {
					// syntax: +vv-t maybe-some targets
					// we want the number of mode changes in the message, so count the
					// number of chars other than + and - before the first space
					const modeChangesCount = message.text
						.split(" ")[0]
						.split("")
						.filter((char) => char !== "+" && char !== "-").length;
					obj[message.type] += modeChangesCount;
				} else {
					obj[message.type]++;
				}
			}

			// Count quits as parts in condensed messages to reduce information density
			obj.part += obj.quit;

			const strings = [];
			constants.condensedTypes.forEach((type) => {
				if (obj[type]) {
					switch (type) {
						case "chghost":
							strings.push(
								obj[type] +
									(obj[type] > 1
										? " users have changed hostname"
										: " user has changed hostname")
							);
							break;
						case "join":
							strings.push(
								obj[type] +
									(obj[type] > 1 ? " users have joined" : " user has joined")
							);
							break;
						case "part":
							strings.push(
								obj[type] + (obj[type] > 1 ? " users have left" : " user has left")
							);
							break;
						case "nick":
							strings.push(
								obj[type] +
									(obj[type] > 1
										? " users have changed nick"
										: " user has changed nick")
							);
							break;
						case "kick":
							strings.push(
								obj[type] +
									(obj[type] > 1 ? " users were kicked" : " user was kicked")
							);
							break;
						case "mode":
							strings.push(
								obj[type] + (obj[type] > 1 ? " modes were set" : " mode was set")
							);
							break;
					}
				}
			});

			let text = strings.pop();

			if (strings.length) {
				text = strings.join(", ") + ", and " + text;
			}

			return text;
		},
	},
	methods: {
		onCollapseClick() {
			this.isCollapsed = !this.isCollapsed;
			this.keepScrollPosition();
		},
	},
};
</script>
