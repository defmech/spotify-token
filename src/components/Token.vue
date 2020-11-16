<template>
	<section>
		<h1>Spotify Token Generator</h1>

		<hr />

		<form submit.prevent>
			<h2>Spotify</h2>
			<div class="form-group row">
				<label for="clientID" class="col-sm-2 col-form-label">Client ID </label>
				<div class="col-sm-10">
					<input
						v-model="clientID"
						class="form-control"
						type="text"
						id="clientID"
						placeholder="Spotify Client ID"
					/>
				</div>
			</div>

			<div class="form-group row">
				<label for="clientSecret" class="col-sm-2 col-form-label">Client Secret </label>
				<div class="col-sm-10">
					<input
						v-model="clientSecret"
						class="form-control"
						type="text"
						id="clientSecret"
						placeholder="Spotify Client Secret"
					/>
				</div>
			</div>

			<div>
				<p class="alert alert-warning">
					<strong>Note:</strong> The Spotify client secret is only used client side and is not used
					by this website except to generate the <strong>curl</strong> string which you execute
					manually.
				</p>
			</div>

			<!-- Step 1 -->
			<hr />

			<h2>Step 1</h2>
			<div class="alert alert-dark" role="alert">
				Set the Redirect URI (edit settings button) on your Spotify app to a url you control.
			</div>

			<div class="form-group row">
				<label for="redirectURI" class="col-sm-2 col-form-label">Redirect URI </label>
				<div class="col-sm-10">
					<input
						v-model="redirectURI"
						class="form-control"
						type="text"
						id="redirectURI"
						placeholder="Redirect URL"
					/>
				</div>
			</div>

			<div class="form-group row">
				<label for="requestScope" class="col-sm-2 col-form-label">Scope</label>
				<div class="col-sm-10">
					<input v-model="requestScope" class="form-control" type="text" id="requestScope" />
				</div>
			</div>

			<div class="alert alert-success" role="alert">
				Open the following link. Authorise using your Spotify account.
			</div>

			<div v-if="codeURL">
				<a :href="codeURL" target="_blank">{{ codeURL }}</a>
			</div>
			<div v-if="!codeURL">Please make sure above fields are complete.</div>

			<hr />

			<div class="form-group row">
				<label for="redirectResult" class="col-sm-2 col-form-label"
					>window.location from Redirect URI</label
				>
				<div class="col-sm-10">
					<textarea class="form-control" v-model="redirectResult" type="text" id="redirectResult" />
				</div>
			</div>

			<hr />

			<!-- Step 2 -->

			<div>
				<h2>Step 2</h2>
				<div><p>Copy and paste the following into a terminal window.</p></div>
				<samp>{{ curl }}</samp>
				<hr />
				<div class="form-group row">
					<label for="curlResult" class="col-sm-2 col-form-label">Curl results</label>
					<div class="col-sm-10">
						<textarea class="form-control" v-model="curlResult" type="text" id="curlResult" />
					</div>
				</div>

				<hr />

				<div v-if="curlResultObject">
					<dl>
						<dt>Access Token</dt>
						<dd>{{ curlResultObject.access_token }}</dd>
						<dt>Expires in</dt>
						<dd>{{ curlResultObject.expires_in / 60 }} mins</dd>
						<dt>Scope</dt>
						<dd>{{ curlResultObject.scope }}</dd>
					</dl>
					<hr />
				</div>

				<div v-if="curlResultObject">
					<h3>Sample Code</h3>
					<samp>{{ codeSnippet }}</samp>
					<hr />
				</div>
			</div>
		</form>
	</section>
</template>

<script>
import base64 from 'base-64';
import queryString from 'query-string';

export default {
	data() {
		return {
			clientID: null,
			clientSecret: null,
			requestScope: 'user-read-email user-read-private',
			redirectURI: null,
			redirectResult: null,
			curlResult: null,
		};
	},
	computed: {
		urlEncodedRedirectURI() {
			return encodeURI(this.redirectURI);
		},

		urlEncodedRequestScope() {
			return encodeURI(this.requestScope);
		},

		codeURL() {
			if (!this.clientID || !this.redirectURI || !this.urlEncodedRequestScope) {
				return null;
			}

			return `https://accounts.spotify.com/authorize?client_id=${this.clientID}&response_type=code&redirect_uri=${this.redirectURI}&scope=${this.urlEncodedRequestScope}`;
		},

		encodedAuth() {
			return base64.encode(`${this.clientID}:${this.clientSecret}`);
		},

		curl() {
			if (!this.accessCode) {
				return null;
			}

			return `curl -H "Authorization: Basic ${this.encodedAuth}" -d grant_type=authorization_code -d code=${this.accessCode} -d redirect_uri=${this.urlEncodedRedirectURI} https://accounts.spotify.com/api/token`;
		},

		curlResultObject() {
			if (!this.curlResult) {
				return null;
			}

			let result = this.curlResult;

			// Strip off the end charcter if it's a percent sign
			if (result[result.length - 1] === '%') {
				result = result.substring(0, result.length - 1);
			}

			return JSON.parse(result);
		},
		accessCode() {
			if (!this.redirectResult) {
				return null;
			}

			const parsed = queryString.parse(this.redirectResult.replace(`${this.redirectURI}/?`, ''));

			const { code } = parsed;
			return code || null;
		},

		codeSnippet() {
			return `spotifyApi.setAccessToken('${this.curlResultObject.access_token}');`;
		},
	},
};
</script>

<style lang="sass" scoped></style>
