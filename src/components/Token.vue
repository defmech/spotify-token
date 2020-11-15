<template>
	<section>
		<form submit.prevent>
			<fieldset>
				<legend>Spotify</legend>

				<div>
					<p>Create an app at https://developer.spotify.com/dashboard</p>
				</div>
				<div>
					<p>
						Note: The Spotify client secret is only used client side and is not used by this
						website except to generate the curl string which you execute manually.
					</p>
				</div>
				<div>
					<label for="clientID"
						>Client ID <input v-model="clientID" type="text" id="clientID"
					/></label>
				</div>
				<div>
					<label for="clientSecret"
						>Client Secret <input v-model="clientSecret" type="text" id="clientSecret"
					/></label>
				</div>
			</fieldset>

			<fieldset>
				<legend>Step 1</legend>
				<div>
					<p>
						Set the Redirect URI (edit settings button) on your Spotify app to a url you control.
					</p>
				</div>

				<div>
					<label for="redirectURI"
						>Redirect URI <input v-model="redirectURI" type="text" id="redirectURI"
					/></label>
				</div>
				<div>
					<label for="requestScope"
						>Scope <input v-model="requestScope" type="text" id="requestScope"
					/></label>
				</div>
				<div>
					<p>
						Open the following link. Authorise using your Spotify account.
					</p>
				</div>
				<div v-if="codeURL"><a :href="codeURL" target="_blank">Open in tab</a></div>
				<div v-if="!codeURL">Please make sure above fields are complete.</div>
				<div>
					<p>
						Copy full URL from redirected page into the following field.
					</p>
				</div>
				<div>
					<textarea rows="4" cols="80" v-model="redirectResult" type="text" id="redirectResult" />
				</div>
				<div v-if="accessCode">{{ accessCode }}</div>
				<div v-if="!accessCode">No access code.</div>
			</fieldset>

			<fieldset>
				<legend>Step 2</legend>
				<div><p>Copy and paste the following into a terminal window.</p></div>
				<pre>{{ curl }}</pre>
				<hr />
				<div><p>Paste the results here.</p></div>
				<div></div>
				<textarea rows="7" cols="80" v-model="curlResult" type="text" id="curlResult" />
				<hr />
				<div>
					<dl>
						<dt>Access Token</dt>
						<dd>{{ curlResultObject.access_token }}</dd>
						<dt>Expires in</dt>
						<dd>{{ curlResultObject.expires_in / 60 }} mins</dd>
						<dt>Scope</dt>
						<dd>{{ curlResultObject.scope }}</dd>
					</dl>
				</div>
				<hr />
				<pre>{{ codeSnippet }}</pre>
			</fieldset>
		</form>
	</section>
</template>

<script>
import base64 from 'base-64';
import queryString from 'query-string';

export default {
	data() {
		return {
			clientID: 'a7aeab5447fb4b86b8d4a628a033723a',
			clientSecret: '017964e16b0a4737869c8693d261b0db',
			requestScope: 'user-read-email user-read-private',
			redirectURI: 'https://defmech.com',
			redirectResult:
				'https://defmech.com/?code=AQAFtkpjQa3cAC07z-jX3mCiE3846U58AeMCAvV-fl_4h6otZO9B8SlbgQSM6C-6Ter-Dbubqp6vgziV1PuIhaImKsrOkqWsR2fcOQdVRCaW3PHf_D32nx-2H0BzRRvfKwjbgWhniuHbVbzkfDlTnVpETCGwtfrIg5OutYQUlviGGx1kI8N30K4Cmr-Xm1q33SXAtlA8PoRRp7k',
			curlResult:
				'{"access_token":"BQB0PtcVwDsKDZsGoRIAF0GtDhrYzzOdu-I8Zk6TeWW34TPBwuA7QPMRlpEHUFOxAEkZ4Nmrlz7eLZA1VFAz3i4cppxgBb_-JpEBRWRxjpuKv6E0EezLUNhaNxcuE-Sdme-xgSZdZ9EY76qiyJu2nnr8YQ","token_type":"Bearer","expires_in":3600,"refresh_token":"AQCliww41rzpTCQAa_bsOfCq32MuLRQN6gfjmMklfCEQZUeouD4dpVkfMr8FLKtv_ZQvd0Mh5EPR0BLhGZpAly0EYRssKzqASVkkUzAgvhpAjrAX9fII_uHags4fQpHmm-c","scope":"user-read-email user-read-private"}%',
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
			return `curl -H "Authorization: Basic ${this.encodedAuth}" -d grant_type=authorization_code -d code=${this.accessCode} -d redirect_uri=${this.urlEncodedRedirectURI} https://accounts.spotify.com/api/token`;
		},

		curlResultObject() {
			let result = this.curlResult;

			// Strip off the end charcter if it's a percent sign
			if (result[result.length - 1] === '%') {
				result = result.substring(0, result.length - 1);
			}

			return JSON.parse(result);
		},
		accessCode() {
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
