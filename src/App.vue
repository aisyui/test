<template>
	<div id="app">
		<div v-if="record && record.data.records[0].uri !== uri" class="bluesky-record">
			<div v-for="i in record.data.records">
				<p><span class="handle">@{{ handle }}</span></p>
				<p><span class="text">{{ i.value.text }}</span></p>
				<p><span class="time"><a :href="i.uri">{{ i.value.createdAt }}</a></span></p>
			</div>
		</div>
	</div>
</template>

<script>
import axios from 'axios'
export default {
	data () {
		return {
			handle: "yui.syui.ai",
			record: null,
		}
	},
	mounted () {
			axios
				.get("https://bsky.social/xrpc/com.atproto.repo.listRecords?repo=" + this.handle + "&collection=app.bsky.feed.post&limit=1")
				.then(response => (this.record = response));
	}
}
</script>

<style>
div#app {
    background-color:rgb(255, 214, 10);
}
</style>
