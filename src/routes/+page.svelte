<script lang="ts">
	// Landing page for pianoboi
</script>

<svelte:head>
	<title>pianoboi - Interactive MIDI Piano</title>
	<meta
		name="description"
		content="Interactive piano with chord detection and sheet music display. Learn to play piano with real-time chord recognition and visualization."
	/>
	<!-- Open Graph / Facebook -->
	<meta property="og:type" content="website" />
	<meta property="og:url" content="https://pianoboi.com/" />
	<meta property="og:title" content="pianoboi - Interactive MIDI Piano" />
	<meta property="og:description" content="Interactive piano with chord detection and sheet music display. Learn to play piano with real-time chord recognition and visualization." />
	<meta property="og:site_name" content="pianoboi" />
	<meta property="og:locale" content="en_US" />
	<!-- Twitter -->
	<meta property="twitter:card" content="summary_large_image" />
	<meta property="twitter:url" content="https://pianoboi.com/" />
	<meta property="twitter:title" content="pianoboi - Interactive MIDI Piano" />
	<meta property="twitter:description" content="Interactive piano with chord detection and sheet music display. Learn to play piano with real-time chord recognition and visualization." />
	<!-- Canonical URL -->
	<link rel="canonical" href="https://pianoboi.com/" />
	<!-- Keywords -->
	<meta name="keywords" content="piano, MIDI, chord detection, sheet music, music learning, interactive piano, learn piano, keyboard online, pianoboi" />
	<!-- Additional SEO tags -->
	<meta name="author" content="pianoboi" />
	<script type="application/ld+json">
	{
		"@context": "https://schema.org",
		"@type": "WebApplication",
		"name": "pianoboi",
		"url": "https://pianoboi.com",
		"description": "Interactive piano with chord detection and sheet music display. Learn to play piano with real-time chord recognition and visualization.",
		"applicationCategory": "MusicApplication",
		"operatingSystem": "Any",
		"offers": {
			"@type": "Offer",
			"price": "0",
			"priceCurrency": "USD"
		},
		"featureList": [
			"MIDI compatibility",
			"Chord detection",
			"Sheet music display",
			"Save and organize chords",
			"Free to use"
		]
	}
	</script>
</svelte:head>

<main class="min-h-screen bg-gray-50">
	<div class="mx-auto max-w-6xl px-4 py-12 sm:px-6 lg:px-8">
		<!-- Hero section -->
		<div class="text-center mb-16">
			<h1 class="text-5xl font-bold text-gray-900 mb-4">pianoboi</h1>
			<p class="text-xl text-gray-600 max-w-3xl mx-auto">
				An interactive MIDI piano with chord detection and sheet music display
			</p>
			<div class="mt-8">
				<a
					href="/app"
					class="inline-block px-6 py-3 bg-blue-500 text-white font-medium rounded-lg shadow-md hover:bg-blue-600 transition-colors"
				>
					🎹 Begin Playing
				</a>
				<p class="mt-2 text-sm text-gray-600">No login required — start playing instantly</p>
			</div>
		</div>

		<!-- Features section -->
		<div class="grid grid-cols-1 md:grid-cols-3 gap-8 mb-16">
			<div class="bg-white p-6 rounded-lg shadow-sm">
				<div class="text-blue-500 mb-3">
					<svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19V6l12-3v13M9 19c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zm12-3c0 1.105-1.343 2-3 2s-3-.895-3-2 1.343-2 3-2 3 .895 3 2zM9 10l12-3" />
					</svg>
				</div>
				<h3 class="text-lg font-medium text-gray-900 mb-2">MIDI Compatible</h3>
				<p class="text-gray-600">
					Connect your MIDI keyboard or piano and play in real-time with instant chord detection
				</p>
			</div>

			<!-- Cost feature card -->
			<div class="bg-white p-6 rounded-lg shadow-sm">
				<div class="text-blue-500 mb-3">
					<svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
					</svg>
				</div>
				<h3 class="text-lg font-medium text-gray-900 mb-2">100% Free</h3>
				<p class="text-gray-600">
					No subscription, no payments, no hidden fees. Just open and play.
				</p>
			</div>

			<div class="bg-white p-6 rounded-lg shadow-sm">
				<div class="text-blue-500 mb-3">
					<svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
						<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2" />
					</svg>
				</div>
				<h3 class="text-lg font-medium text-gray-900 mb-2">Save & Organize</h3>
				<p class="text-gray-600">
					Save your chords and organize them into sets for practice or composition
				</p>
			</div>
		</div>

		<!-- FAQ section -->
		<div class="mb-16">
			<h2 class="text-3xl font-bold text-gray-900 mb-8 text-center">Frequently Asked Questions</h2>
			
			<div class="space-y-6 max-w-4xl mx-auto">
				<!-- Cost/pricing FAQ - placed first to make it prominent -->
				<div class="bg-white p-6 rounded-lg shadow-sm border-l-4 border-blue-500">
					<h3 class="text-lg font-medium text-gray-900 mb-2">Is pianoboi free to use?</h3>
					<p class="text-gray-600">
						Yes, pianoboi is completely free to use. There are no subscriptions, no premium tiers, and no hidden fees. All features are available to everyone without any cost. You don't even need to create an account to use the app.
					</p>
				</div>
				
				<div class="bg-white p-6 rounded-lg shadow-sm border-l-4 border-blue-500">
					<h3 class="text-lg font-medium text-gray-900 mb-2">Why can't I hear sound when I play?</h3>
					<p class="text-gray-600">
						The current version has basic sound playback that loads piano samples in the browser. This is meant as a reference but doesn't fully replace a proper MIDI output device. For the best experience, we recommend using an external MIDI instrument or synthesizer. We plan to improve browser-based sound synthesis in future updates.
					</p>
				</div>
				
				<div class="bg-white p-6 rounded-lg shadow-sm border-l-4 border-blue-500">
					<h3 class="text-lg font-medium text-gray-900 mb-2">I can't play chords by clicking with my mouse. Is this a bug?</h3>
					<p class="text-gray-600">
						pianoboi is primarily designed for hardware MIDI input from a piano keyboard or MIDI controller. While you can click piano keys with your mouse, it's difficult to play multiple notes simultaneously. For the best experience, connect a MIDI keyboard. If you don't have a MIDI device, you can use your computer keyboard with MIDI emulation software, though this isn't directly supported in the app.
					</p>
				</div>
				
				<div class="bg-white p-6 rounded-lg shadow-sm border-l-4 border-blue-500">
					<h3 class="text-lg font-medium text-gray-900 mb-2">Which browsers are supported?</h3>
					<p class="text-gray-600">
						pianoboi works best in Chrome and Firefox, which fully support the WebMIDI API. Safari has limited support for WebMIDI due to security and privacy considerations. For the best experience, we recommend using the latest version of Chrome or Firefox. If you're having compatibility issues, please try updating your browser.
					</p>
				</div>
				
				<div class="bg-white p-6 rounded-lg shadow-sm border-l-4 border-blue-500">
					<h3 class="text-lg font-medium text-gray-900 mb-2">Do I need a MIDI keyboard to use pianoboi?</h3>
					<p class="text-gray-600">
						While pianoboi is optimized for use with a MIDI keyboard or controller, you can still use the on-screen keyboard by clicking with your mouse. This is useful for exploring the app's features, but for a full experience, a MIDI input device is recommended. Both USB-MIDI and traditional MIDI connections (via an interface) are supported.
					</p>
				</div>
				
				<div class="bg-white p-6 rounded-lg shadow-sm border-l-4 border-blue-500">
					<h3 class="text-lg font-medium text-gray-900 mb-2">How do I save my chords?</h3>
					<p class="text-gray-600">
						Press the Space bar to save the currently played chord. You can organize saved chords into sets using the Set dropdown menu. Chords are automatically saved in your browser's local storage, so they will persist between sessions on the same device and browser.
					</p>
				</div>
			</div>
		</div>

		<footer class="text-center text-sm text-gray-500 pt-8 border-t">
			<p class="mt-2">© 2025 pianoboi</p>
		</footer>
	</div>
</main>
