<template>
	<div id="app">
		<Token />
	</div>
</template>

<script>
import Token from './components/Token.vue';

export default {
	name: 'App',
	components: {
		Token,
	},
};
</script>

<style lang="scss">
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
