<!-- Weather icons: https://erikflowers.github.io/weather-icons/-->


<script>
	import { onMount } from 'svelte';

	/****SETTINGS****/
	
	//Other options are 'metric' (for celsius) and 'standard' (for kelvin)
	const tempUnit = 'imperial'; 
	
	//Make true to display weather information for each city
	var showWeather = false;

	//These values are used to indicate color of time bar and position of work day indicator
	//Program does not recognize a range that spans past midnight so values here must run sequentially 
	const dayStart = 6;
	const workStart = 8;
	const workEnd = 17;
	const dayEnd = 24;

	//API keys here. Only need open weather API if using weather feature
	const timeZoneDbKey = '2RZSAN6G8BW6';
	const openWeatherMapKey = '0e126e0bf1f132591bd57cd3441ce754';

	/*Create, delete and reorder cities that are shown.
	To add new time zone, just add a new line with same values except the name of the city and the zoneName as found at https://timezonedb.com/time-zones
	Name of city is the title that is displayed, but is also used to pull weather data, so it's better to choose larger cities if using the weather feature*/
	var locations = [
		{name:'Seattle', zoneName: 'America/Los_Angeles', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Calgary', zoneName: 'America/Edmonton', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Chicago', zoneName: 'America/Chicago', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'New York', zoneName: 'America/New_York', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'London', zoneName: 'Europe/London', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Johannesburg', zoneName: 'Africa/Johannesburg', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Athens', zoneName: 'Europe/Athens', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Kolkata', zoneName: 'Asia/Kolkata', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Manila', zoneName: 'Asia/Manila', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Shanghai', zoneName: 'Asia/Shanghai', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Tokyo', zoneName: 'Asia/Tokyo', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0},
		{name:'Sydney', zoneName: 'Australia/Sydney', timeZoneOffset: 0, time: '', hours: 0, weatherIcon: 'na', temp: 0, tileColor: '#747474', workdayPerc: 0}
	];

	/****END SETTINGS****/

	const timeZoneApiUrl = 'http://api.timezonedb.com/v2.1/list-time-zone?key=' + timeZoneDbKey +'&format=json';
	var weatherApiUrl = 'http://api.openweathermap.org/data/2.5/weather?q={city}&appid=' + openWeatherMapKey + '&units=' + tempUnit;

	var  userOffset, timeZoneList;
	var sliderPos = 0;
	let sliderColor = '#747474';

	$: if(sliderPos > 0) {
		sliderColor = '#FF652F';
	} else {
		sliderColor = '#747474';
	}

	var percentColors = [
		{ pct: 0.0, color: { r: 0x14, g: 0xa7, b: 0x6c } },
		{ pct: 0.5, color: { r: 0xff, g: 0xe4, b: 0x00 } },
		{ pct: 1.0, color: { r: 0xff, g: 0x65, b: 0x25 } } ];

	$: sliderPos && getTime();

	function getTime() {
		for(var i = 0; i < locations.length; i++) {
			var now = new Date();
			var adjustedDate = new Date();
			adjustedDate.setMinutes(now.getMinutes() + (locations[i].timeZoneOffset - userOffset) * 60 + sliderPos);

			locations[i].tileColor = getTileColor(adjustedDate);
			locations[i].workdayPerc = getWorkdayPerc(adjustedDate);

			locations[i].hours = adjustedDate.getHours();
			locations[i].time = formatAMPM(adjustedDate);
		}

		locations = locations;
	}

	function getTileColor(date) {
		if(checkIfWorking(date)) {
			return '#14A76C';
		} else if(date.getHours() < dayStart || date.getHours() == dayStart && date.getMinutes() == 0) {
			return '#FF652F';
		} else {
			return getColorForPercentage(getPercentage((date.getHours() * 60) + date.getMinutes()));
		}
	}

	function getPercentage(minutes) {
		if(minutes < (dayEnd * 60) && minutes > (workEnd * 60)) {
			return ((minutes - (workEnd * 60)) / ((dayEnd - workEnd) * 60));
		} else {
			return 1 - ((minutes - (dayStart * 60)) / ((workStart - dayStart) * 60));
		}
	}

	function checkIfWorking(date) {
		return (date.getHours() > (workStart - 1) && date.getHours() < workEnd) || (date.getHours() == workEnd && date.getMinutes() == 0)
	}

	function getColorForPercentage(pct) {
		for (var i = 1; i < percentColors.length - 1; i++) {
			if (pct < percentColors[i].pct) {
				break;
			}
		}
		var lower = percentColors[i - 1];
		var upper = percentColors[i];
		var range = upper.pct - lower.pct;
		var rangePct = (pct - lower.pct) / range;
		var pctLower = 1 - rangePct;
		var pctUpper = rangePct;
		var color = {
			r: Math.floor(lower.color.r * pctLower + upper.color.r * pctUpper),
			g: Math.floor(lower.color.g * pctLower + upper.color.g * pctUpper),
			b: Math.floor(lower.color.b * pctLower + upper.color.b * pctUpper)
		};
		return 'rgb(' + [color.r, color.g, color.b].join(',') + ')';
	};

	function getWorkdayPerc(date) {
		if(checkIfWorking(date)) {
			var minutes = date.getHours() * 60 + date.getMinutes();
			return ((minutes - (workStart * 60)) / ((workEnd - workStart) * 60));;
		} else {
			return 0;
		}
	}

	function formatAMPM(date) {
		var hours = date.getHours();
		var minutes = date.getMinutes();
		var ampm = hours >= 12 ? 'pm' : 'am';
		hours = hours % 12;
		hours = hours ? hours : 12; // the hour '0' should be '12'
		minutes = minutes < 10 ? '0'+ minutes : minutes;
		var strTime = hours + ':' + minutes + ' ' + ampm;
		return strTime;
	}

	function extractTimeZones() {
		var loc;
		for (loc of locations) {
			loc.timeZoneOffset = timeZoneList.zones.find(zone => {
				return zone.zoneName === loc.zoneName;
			}).gmtOffset / (60 * 60);
		}

		locations = locations;
		setInterval(getTime, 1000);
	}

	async function getWeatherData() {
		var loc;
		for(loc of locations) {
			var url = weatherApiUrl.replace('{city}', loc.name);
			const response = await fetch(url);
				await response.json()
				.then(res => {
					console.log(res);
					loc.weatherIcon = res.weather[0].icon;
					loc.temp = Math.round(res.main.temp);
				});
		}

		locations = locations;
	}

	onMount(async () =>	{
		const response = await fetch(timeZoneApiUrl);
		await response.json()
		 .then(res => {
			 console.log(res);
			 timeZoneList = res;
			 extractTimeZones();
		 });

		 if (showWeather) { 
			getWeatherData();
		 	setInterval(getWeatherData, 900000); //Check every 15 minutes
		 }
		 
		 userOffset = (new Date().getTimezoneOffset() / 60) * -1;
	})
</script>

<main>
	<div class='slidecontainer'>
		<input type='range' min='0' max='1440' bind:value={sliderPos} class='slider' style='background:{sliderColor}'>
	</div>
	<div class='locations-container'>
		{#each locations as location}
			<div class='location'>
				<div class='location-text'>
					<div class='time'>{location.time}</div>
					<div class='city-name'>{#if location.timeZoneOffset >= 0 }+{/if}{location.timeZoneOffset} | {location.name}
						{#if showWeather}<img class='weather-icon' src='./images/{location.weatherIcon}.svg' alt='Weather Icon'/>/ {location.temp}&#176;{/if}
					</div>
					<div class='work-perc-ind-container'>
						<div class='work-perc-ind' style='left:{location.workdayPerc * 100}%'></div>
					</div>
				</div>
				<div class='time-band' style='background-color:{location.tileColor}'></div>
			</div>
		{/each}
	</div>
</main>

<style>
	:root {
		--green:#14A76C;
		--yellow:#FFE400;
		--red:#FF652F;
		--gray:#747474;
		--black: #272727;	
		background-color: var(--black);
	}

	.locations-container {
		display: flex;
		flex-direction: row;
		align-items: stretch;
		flex-wrap: wrap;
		flex: 1;
		overflow: auto;
		height: 100%;
	}

	.location {
		position: relative;
		flex-grow: 1;
		background-color: var(--gray);
		flex-basis: 150pt;
		color: white;
		padding: 8pt;
		margin: 4pt;
		box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
  		transition: 0.3s;
		min-height: 68pt;
		overflow: hidden;
	}

	.location-text {
		margin-left:10%;
	}

	.weather-icon {
		height: 24pt;
		-webkit-filter: invert(100%);
		vertical-align: -50%;
		padding-left: 4pt;
	}

	.time {
		font-size: 32pt;
	}

	.work-perc-ind-container {
		position: absolute;
		bottom: 0;
		left:0;
		/*Accounting for 8pt of indicator so it disspears on either side*/
		width: calc(90% + 8pt);
		margin-left:calc(10% - 8pt);
	}

	.work-perc-ind {
		position: absolute;
		bottom: 0;
		height: 4pt;
		width: 8pt;
		background-color: white;	
	}

	.time-band {
		position: absolute;
		top: 0;
		left: 0;
		height: 100%;
		width: 10%;
	}

	.slidecontainer {
		width: 100%; 
	}

	.slider {
		-webkit-appearance: none; 
		appearance: none;
		width: 100%; 
		height: 8px; 
		outline: none;
		opacity: 0.7; 
		-webkit-transition: .2s; 
		transition: opacity .2s;
	}

	.slider:hover {
		opacity: 1; 
	}

	.slider::-webkit-slider-thumb {
		-webkit-appearance: none; 
		appearance: none;
		width: 12px; 
		height: 12px; 
		background: white; 
		cursor: pointer; 
	}

	.slider::-moz-range-thumb {
		width: 12px; 
		height: 12px; 
		background: white; 
		cursor: pointer;
	}
</style>