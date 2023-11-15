<template>
	<div id="app">

		<form @submit.prevent="login">
			<p><input v-model="handle" placeholder="user.bsky.social"></p>
			<p><input v-model="password" placeholder="password"></p>
			<button type="login">send</button>
		</form>

		<div v-if="record_login !== null">
			login : true
			<form @submit.prevent="post">
				<p><input v-model="text" placeholder="text"></p>
				<button type="post">post</button>
			</form>
		</div>

		<button @click="option='&limit=100'">limit</button>
		<button @click="option='&reverse=true'">revert</button>
		<div v-if="record" class="bluesky-record">
			<button @click="cursor(record.data.cursor)">next</button>
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
import moment from 'moment'
export default {
	data () {
		return {
			api: "https://bsky.social",
			handle: "user.bsky.social",
			password: null,
			record: null,
			record_login: null,
			option: "&limit=1",
			time: moment.utc().toISOString(),
			json: null,
			text: null,
		}
	},
	methods: {
		cursor(s){
			this.option = "&limit=1&cursor=" + s;
		},
		login() {
			axios
				.post(this.api + "/xrpc/com.atproto.server.createSession", {
					identifier: this.handle,
					password: this.password,
				})
				.then(response => (this.record_login = response));
		},
		moment: function (date) {
			return moment.unix(date).toISOString()
		},
		post() {
			this.json = {
				did: this.record_login.data.did,
				repo: this.record_login.data.handle,
				collection: "app.bsky.feed.post",
				record: {
					text: this.text,
					createdAt: this.time,
					'$type': "app.bsky.feed.post"
				}
			};
			axios
				.post(this.api + "/xrpc/com.atproto.repo.createRecord", this.json, {
					headers: {
						Authorization: "Bearer " + this.record_login.data.accessJwt
					},
				})
				.then(response => (this.login_post = response));
		},
	},
	mounted () {
		axios
			.get(this.api + "/xrpc/com.atproto.repo.listRecords?repo=" + this.handle + "&collection=app.bsky.feed.post" + this.option)
			.then(response => (this.record = response));
	},
	updated () {
		axios
			.get(this.api + "/xrpc/com.atproto.repo.listRecords?repo=" + this.handle + "&collection=app.bsky.feed.post" + this.option)
			.then(response => (this.record = response));
	},
}
</script>

<style>
div#app {
    background-color:rgb(255, 214, 10);
}
</style>
