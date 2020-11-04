<template>
<div class="w-screen h-screen flex-col bg-teal-700 overflow-hidden">
	<!--nav bar-->
	<div class="flex-1 h-14 bg-pink-600">
		<div class="flex-1 h-14 px-2 flex items-center bg-gray-900">
			<div class="w-auto h-12 flex items-center justify-center rounded-md cursor-pointer select-none">
				<div class="w-10 h-12 flex items-center justify-center">
					<svg class="transform scale-150 text-pink-700" width="24" height="24" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path
							fill="currentColor"
							d="M7.25 12a.75.75 0 000 1.5h.5a.75.75 0 000-1.5h-.5zM6.5 9.25a.75.75 0 01.75-.75h.5a.75.75 0 010 1.5h-.5a.75.75 0 01-.75-.75zM7.25 5a.75.75 0 000 1.5h.5a.75.75 0 000-1.5h-.5zM10 12.75a.75.75 0 01.75-.75h.5a.75.75 0 010 1.5h-.5a.75.75 0 01-.75-.75zM10.75 8.5a.75.75 0 000 1.5h.5a.75.75 0 000-1.5h-.5zM10 5.75a.75.75 0 01.75-.75h.5a.75.75 0 010 1.5h-.5a.75.75 0 01-.75-.75zM14.25 12a.75.75 0 000 1.5h.5a.75.75 0 000-1.5h-.5zM13.5 9.25a.75.75 0 01.75-.75h.5a.75.75 0 010 1.5h-.5a.75.75 0 01-.75-.75zM14.25 5a.75.75 0 000 1.5h.5a.75.75 0 000-1.5h-.5z"
						/>
						<path
							fill-rule="evenodd"  clip-rule="evenodd"
							fill="currentColor"
							d="M3 20a2 2 0 002 2h3.75a.75.75 0 00.75-.75V19h3v2.25c0 .414.336.75.75.75H17c.092 0 .183-.006.272-.018a.758.758 0 00.166.018H21.5a2 2 0 002-2v-7.625a2 2 0 00-.8-1.6l-1-.75a.75.75 0 10-.9 1.2l1 .75a.5.5 0 01.2.4V20a.5.5 0 01-.5.5h-2.563c.041-.16.063-.327.063-.5V3a2 2 0 00-2-2H5a2 2 0 00-2 2v17zm2 .5a.5.5 0 01-.5-.5V3a.5.5 0 01.5-.5h12a.5.5 0 01.5.5v17a.5.5 0 01-.5.5h-3v-2.25a.75.75 0 00-.75-.75h-4.5a.75.75 0 00-.75.75v2.25H5z"
						/>
					</svg>
				</div>
				<div class="h-12 px-1 flex items-center justify-center flex-1">
					<h1 class="text-gray-100 font-semibold text-4xl">Yeah</h1>
				</div>
			</div>
			<div class="ml-auto">
				<button class="w-10 h-10 px-1 focus:outline-none flex items-center justify-center rounded-full overflow-hidden text-gray-200 text-lg bg-gray-900">
					<a class="w-10 h-10 flex items-center justify-center" href="https://github.com/SOKHUONG/VTour" target="_blank">
						<svg width="24" height="24" fill="none" xmlns="http://www.w3.org/2000/svg">
							<path fill-rule="evenodd" clip-rule="evenodd" d="M12.5.75C6.146.75 1 5.896 1 12.25c0 5.089 3.292 9.387 7.863 10.91.575.101.79-.244.79-.546 0-.273-.014-1.178-.014-2.142-2.889.532-3.636-.704-3.866-1.35-.13-.331-.69-1.352-1.18-1.625-.402-.216-.977-.748-.014-.762.906-.014 1.553.834 1.769 1.179 1.035 1.74 2.688 1.25 3.349.948.1-.747.402-1.25.733-1.538-2.559-.287-5.232-1.279-5.232-5.678 0-1.25.445-2.285 1.178-3.09-.115-.288-.517-1.467.115-3.048 0 0 .963-.302 3.163 1.179.92-.259 1.897-.388 2.875-.388.977 0 1.955.13 2.875.388 2.2-1.495 3.162-1.179 3.162-1.179.633 1.581.23 2.76.115 3.048.733.805 1.179 1.825 1.179 3.09 0 4.413-2.688 5.39-5.247 5.678.417.36.776 1.05.776 2.128 0 1.538-.014 2.774-.014 3.162 0 .302.216.662.79.547C20.709 21.637 24 17.324 24 12.25 24 5.896 18.854.75 12.5.75z"
							fill="currentColor"/>
						</svg>
					</a>
				</button>
			</div>
		</div>
	</div>

	<!--main section-->
	<div class="h-full w-full p-2 overflow-y-auto">
		<div class="flex flex-wrap -mb-4">
			<div class="w-full sm:w-1/2 lg:w-1/3 xl:w-1/4 mb-4 h-90">
				<div class="w-full h-full p-2">
					<div class="w-full h-full rounded overflow-hidden shadow hover:shadow-xl">
						<nuxt-link class="w-full h-full" to="/tours/gameroom">
							<img class="w-full h-full object-cover transform transition-all duration-500 hover:scale-125" src="img/gameroom.jpg" alt="">
						</nuxt-link>
					</div>
				</div>
			</div>

			<div class="w-full sm:w-1/2 lg:w-1/3 xl:w-1/4 mb-4 h-90">
				<div class="w-full h-full p-2">
					<div class="relative w-full h-full overflow-hidden shadow hover:shadow-xl">
						<iframe class="w-full h-full" style="width: 100%; height: 100%;" src="https://nokor.netlify.app/tours/a/" frameborder="0" allowfullscreen allow="vr; xr-spatial-tracking"></iframe>
					</div>
				</div>
			</div>

			<div class="w-full sm:w-1/2 lg:w-1/3 xl:w-1/4 mb-4 h-90">
				<div class="w-full h-full p-2">
					<div class="relative w-full h-full overflow-hidden shadow hover:shadow-xl">
						<iframe id="360_180" src="/tours/360_180" class="w-full h-full"></iframe>
						<button @click="openFullscreen('360_180')" class="absolute right-0 top-0 w-8 h-8 bg-green-800 text-gray-100">
							full
						</button>
					</div>
				</div>
			</div>
			
		</div>
	</div>
</div>
</template>

<script>

import _ from 'lodash';

export default {

	name: 'Home',

	methods: {

		Log(){

			const link = 'https://github.com/SOKHUONG/VTour';

			console.log( [
				'   ___  ___   _____   __   __',
				'  / __\\/\\__\\/ _____\\ / __\\/\\__\\',
				' / /  / /  / /  ___// /  / ___/',
				'/ /  /_/  /  \\__  \\/ /  /  \\',
				'\\/_______/\\/______/\\/__/\\___\\',
			].join( '\n' ) );
			console.log("%c💝 I am really appreciate that you interested in my code 😊", "background: transparent; color: #ed0e90; font-size: 30px; font-weight: 600; font-family: Lobster, serif; height: 200px");
			console.log(`%c Source code of this project on github: %c${link}`,"font-family: Lobster, serif;color: #0eed98; font-size: 25px","font-size: 15px;");

		},

		openFullscreen(id) {

			let iframe = document.getElementById(id);

			if (iframe.requestFullscreen) {

				iframe.requestFullscreen();

			} else if (iframe.mozRequestFullScreen) { /* Firefox */

				iframe.mozRequestFullScreen();

			} else if (iframe.webkitRequestFullscreen) { /* Chrome, Safari & Opera */

				iframe.webkitRequestFullscreen();

			} else if (iframe.msRequestFullscreen) { /* IE/Edge */

				iframe.msRequestFullscreen();

			}

		}

	},

    mounted() {

		if(!sessionStorage.getItem('logged')){

			// _.once(this.Log) will return a funtion, so just call it directly.
			_.once(this.Log)();
			sessionStorage.setItem('logged', true);

		}

		window.onunload = function () {

			sessionStorage.removeItem('logged');

    	}

    }

}

</script>
