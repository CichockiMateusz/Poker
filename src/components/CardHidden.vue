<template>
	<div class="inline">
		<div v-for="(item, index) in deck" :key="index" class="inline-block">
			<div class="w-16 h-20 border-solid border-2 border-black rounded-md grid grid-cols-3 grid-rows-3 text-center bg-white select-none" :class="{ 'text-red-500': item.isRed, 'bg-slate-500': item.selected}" @click="$emit('someEvent', index)">
				<div></div>
				<div></div>
				<div></div>
				<div></div>
				<div></div>
				<div></div>
				<div></div>
				<div></div>
				<div></div>
			</div>
		</div>
    </div>
</template>

<script>
export default {
	props: ['deck']
}
</script>

<style>

</style>