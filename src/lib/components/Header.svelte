<script lang="ts">
	// Header — the site's picker controls come from Lily's picker-bar
	// helper, which composes theme, locale, text-size, and share pickers
	// into one row. Each picker still persists its own slug to
	// localStorage (see the storageKey below and app.html, which applies
	// the stored theme/text-size before first paint to avoid a flash).
	import { ContainerWithFixedWidth, Header } from '@lilydesignsystem/svelte-headless';
	import PickerBar from '@lilydesignsystem/svelte-picker-bar';
	import type { ShareTarget } from '@lilydesignsystem/svelte-share-picker';
	import { themes, DEFAULT_THEME_ID } from '$lib/data/themes';

	const THEME_STORAGE_KEY = 'adr-theme';
	const LOCALE_STORAGE_KEY = 'adr-locale';
	const TEXT_SIZE_STORAGE_KEY = 'adr-text-size';
	const DEFAULT_TEXT_SIZE = 'default';

	const themeSlugs = themes.map((t) => t.id);
	const themeLabels = Object.fromEntries(themes.map((t) => [t.id, t.label]));

	// This site's own 5-step scale (theme.css only defines --user-font-scale
	// for these four data-text-size values; unset/'default' is scale 1) —
	// not picker-bar's 7-step DEFAULT_SIZES.
	const TEXT_SIZES = ['small', 'default', 'large', 'larger', 'largest'];

	// English-only content today; en_US is offered because the picker's
	// built-in label table already distinguishes it ("English" vs
	// "English (United States)"). Selecting either only sets lang/dir on
	// <html> — no translated content exists yet.
	const LOCALES = ['en', 'en_US'];

	const shareTargets: ShareTarget[] = [
		{
			id: 'email',
			label: 'Share on Email',
			href: (url, title) => `mailto:?subject=${encodeURIComponent(title)}&body=${encodeURIComponent(url)}`,
			newTab: false
		},
		{
			id: 'linkedin',
			label: 'Share on LinkedIn',
			href: (url) => `https://www.linkedin.com/sharing/share-offsite/?url=${encodeURIComponent(url)}`
		},
		{
			id: 'reddit',
			label: 'Share on Reddit',
			href: (url, title) => `https://www.reddit.com/submit?url=${encodeURIComponent(url)}&title=${encodeURIComponent(title)}`
		},
		{
			id: 'bluesky',
			label: 'Share on Bluesky',
			href: (url, title) => `https://bsky.app/intent/compose?text=${encodeURIComponent(`${title} ${url}`)}`
		},
		{
			id: 'mastodon',
			label: 'Share on Mastodon',
			href: (url, title) => `https://mastodonshare.com/?text=${encodeURIComponent(title)}&url=${encodeURIComponent(url)}`
		}
	];

	// The closed theme control is one glyph, so nothing else on the page
	// states the active theme. Tracked here (via themeProps.onChange) only
	// to feed the visually-hidden status paragraph below.
	let activeTheme = $state(DEFAULT_THEME_ID);
</script>

<Header label="Site header">
	<ContainerWithFixedWidth maxWidth="64rem">
		<div class="site-header-bar">
			<a class="brand" href="/">
				<span aria-hidden="true">📐</span>
				<strong>Architecture Decision Record</strong>
			</a>
			<nav class="site-nav" aria-label="Primary">
				<a href="/guide/">Guide</a>
				<a href="/templates/">Templates</a>
				<a href="/examples/">Examples</a>
				<a href="/skills/">Claude skills</a>
			</nav>
			<div class="site-header-controls">
				<PickerBar
					labels={{
						theme: 'Colour theme — all Lily Design System themes',
						locale: 'Language',
						textSize: 'Text size',
						share: 'Share Picker'
					}}
					themesUrl="/themes/"
					themes={themeSlugs}
					themeProps={{
						themeLabels,
						storageKey: THEME_STORAGE_KEY,
						defaultValue: DEFAULT_THEME_ID,
						name: 'theme',
						onChange: (theme: string) => (activeTheme = theme)
					}}
					locales={LOCALES}
					localeProps={{ storageKey: LOCALE_STORAGE_KEY, defaultValue: 'en', name: 'locale' }}
					sizes={TEXT_SIZES}
					textSizeProps={{
						storageKey: TEXT_SIZE_STORAGE_KEY,
						defaultValue: DEFAULT_TEXT_SIZE,
						name: 'text-size'
					}}
					{shareTargets}
					shareProps={{
						title: 'Architecture Decision Record (ADR)',
						copyLabel: 'Copy link',
						copiedLabel: 'Link copied to clipboard',
						copyFailedLabel: 'Could not copy — copy it from the address bar'
					}}
				/>
				<a
					class="icon-button"
					href="https://github.com/architecture-decision-record/architecture-decision-record"
					aria-label="View source on GitHub"
				>
					<svg viewBox="0 0 16 16" width="18" height="18" aria-hidden="true" fill="currentColor">
						<path
							d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.01 8.01 0 0 0 16 8c0-4.42-3.58-8-8-8Z"
						/>
					</svg>
				</a>
			</div>
		</div>
	</ContainerWithFixedWidth>
</Header>
<!-- Visually hidden — the header stays icon-only — but present for
     assistive technology, and aria-live announces only on change, so it
     stays silent at first paint. -->
<p class="theme-picker-status visually-hidden" aria-live="polite">
	Active theme: {themeLabels[activeTheme] ?? activeTheme}
</p>
