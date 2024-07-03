<script lang="ts">
	import { onMount } from 'svelte';
	import { io } from 'socket.io-client';
	let in_room: boolean = false;
	let user_room_id: string = '';
	let user_list: any = {};
	let socket: any = undefined;
	onMount(async () => {
		socket = io('http://localhost:3000', {
			reconnection: true,
			reconnectionDelay: 500,
			reconnectionAttempts: 10
		});
		socket.on('connect', () => {
			console.log(`connect ${socket.id}`);
			socket.emit('join');
		});
		socket.on('room', ({ room_id, users }: any) => {
			create_room({ room_id, users });
		});
		socket.on('update', ({ id, msg }: any) => {
			user_list[id] = msg;
		});
		socket.on('room_closed', async () => {
			in_room = false;
			user_room_id = '';
			user_list = {};
			await socket.emit('join');
		});
	});
	function create_room({ room_id, users }: any) {
		in_room = true;
		console.log(users, room_id);
		user_room_id = room_id;
		for (const item of users) {
			user_list[item] = '';
		}
	}
</script>

<main>
	<div class="flex flex-col gap-8 py-8">
		{#if Object.keys(user_list).length > 0}
			<div class="flex flex-col gap-2">
				<h1 class="text-slate-950">{user_room_id}</h1>
				{#each Object.keys(user_list) as id, i}
					<p class="text-slate-950">{id}</p>
					<input
						type="text"
						bind:value={user_list[id]}
						on:input={async () => {
							await socket.emit('update', {
								room_id: user_room_id,
								id,
								msg: user_list[id]
							});
						}}
					/>
				{/each}
			</div>
		{/if}
	</div>
</main>
