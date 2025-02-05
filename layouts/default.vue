<template>
	<main>
		<header>
			<div v-if="$store.state.connectedAddress" class="trezor-status-container">
				{{ $t("trezor_is") }}&nbsp;
				<span
					:class="{
						'trezor-status': true,
						'trezor-status--connected': trezorConnected === true,
						'trezor-status--disconnected': trezorConnected === false,
					}"
					>{{
						trezorConnected ? $tUpper("connected") : $tUpper("disconnected")
					}}</span
				>
			</div>
			<div
				v-if="$store.state.connectedAddress"
				class="address-number-container"
			>
				<span :title="$store.state.connectedAccountPath">
					{{ $tCap("address") }}&nbsp;<span class="address-number">{{
						getCurrentAddressNum()
					}}</span>
				</span>
			</div>

			<a
				class="help-translate"
				href="https://localazy.com/p/briskett"
				rel="noopener"
				target="_blank"
			>
				🌎&nbsp;{{ $tCap("help_translate") }}
			</a>
			<select class="select language-switcher" @change="onLocaleChange">
				<option
					v-for="locale in $i18n.locales"
					:key="locale.code"
					:selected="locale.code === $i18n.locale"
					:value="locale.domain"
				>
					{{ localeEmoji(locale.code) }}&nbsp;{{ locale.code.toUpperCase() }}
				</option>
			</select>

			<div class="logo-container">
				<nuxt-img src="/images/logo.svg" alt="Briskett" />
				<span class="beta-text">[{{ $t("beta") }}]</span>
			</div>

			<h3 v-if="$store.state.connectedAddress" class="balance is-size-3">
				{{ $tCap("balance") }}: {{ $store.state.connectedAddressBalance }}
			</h3>
			<h4 v-if="$store.state.connectedAddress" class="is-size-4">
				{{ $tCap("address") }}: {{ $store.state.connectedAddress }}
			</h4>
			<h4 v-if="$store.state.connectedAddressBaker" class="is-size-4">
				{{ $tCap("baker") }}:
				<a
					rel="noopener"
					target="_blank"
					:href="`https://tzstats.com/${$store.state.connectedAddressBaker}`"
					>{{ $store.state.connectedAddressBaker }}</a
				>
			</h4>

			<div v-if="$store.state.connectedAddress" class="tabs is-centered">
				<ul>
					<li>
						<NuxtLink to="/send">{{ $tCap("send") }}</NuxtLink>
					</li>
					<li>
						<NuxtLink to="/delegate">{{ $tCap("delegate") }}</NuxtLink>
					</li>
          <li>
              <!-- todo: enable when undelegation has been figured out -->
              <!-- <NuxtLink to="/undelegate">{{ $tCap("undelegate") }}</NuxtLink> -->
          </li>
					<li>
						<NuxtLink to="/receive">{{ $tCap("receive") }}</NuxtLink>
					</li>
				</ul>
			</div>
		</header>
		<section>
			<transition name="transition-main">
				<Nuxt />
			</transition>
		</section>
		<section
			v-if="$store.state.connectedAccountOperations.length > 0"
			class="operations"
		>
			<h3 class="is-size-3">
				{{
					$tc("last_operations", $store.state.connectedAccountOperations.length)
				}}
			</h3>
			<div
				v-for="(operation, operationIndex) in $store.state
					.connectedAccountOperations"
				:key="operation.hash + '_' + operation.type"
				class="operation"
			>
				<div v-if="operationIndex === 0" class="header">
					{{ $tCap("type") }}
				</div>
				<div v-if="operationIndex === 0" class="header">
					{{ $tCap("amount") }}
				</div>
				<div v-if="operationIndex === 0" class="header">
					{{ $tCap("time") }}
				</div>
				<div v-if="operationIndex === 0" class="header">
					{{ $tCap("address") }}
				</div>
				<div v-if="operationIndex === 0" class="header">
					{{ $tCap("hash") }}
				</div>
				<!-- Type -->
				<span
					v-if="
						operation.type === 'transaction' &&
						operation.sender === $store.state.connectedAddressBaker
					"
					class="reward"
					>{{ $tCap("reward") }}</span
				>
				<span
					v-else-if="
						operation.type === 'transaction' &&
						operation.sender !== $store.state.connectedAddress
					"
					class="receive"
					>{{ $tCap("receive") }}</span
				>
				<span
					v-else-if="
						operation.type === 'transaction' &&
						operation.sender === $store.state.connectedAddress
					"
					class="send"
					>{{ $tCap("send") }}</span
				>
				<span v-else-if="operation.type === 'delegation'">{{
					$tCap("delegation")
				}}</span>
				<span
					v-else-if="operation.type === 'reveal'"
					class="reveal"
					:aria-label="$t('reveal_operation_details')"
					:title="$t('reveal_operation_details')"
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						role="img"
						viewBox="0 0 512 512"
					>
						<path
							fill="currentColor"
							d="M256 8C119.043 8 8 119.083 8 256c0 136.997 111.043 248 248 248s248-111.003 248-248C504 119.083 392.957 8 256 8zm0 110c23.196 0 42 18.804 42 42s-18.804 42-42 42-42-18.804-42-42 18.804-42 42-42zm56 254c0 6.627-5.373 12-12 12h-88c-6.627 0-12-5.373-12-12v-24c0-6.627 5.373-12 12-12h12v-64h-12c-6.627 0-12-5.373-12-12v-24c0-6.627 5.373-12 12-12h64c6.627 0 12 5.373 12 12v100h12c6.627 0 12 5.373 12 12v24z"
						/>
					</svg>
					{{ $tCap("reveal") }}</span
				>
				<!-- Amount -->
				<div class="amount">{{ operation.volume }}</div>
				<!-- Time -->
				<div>{{ operation.time }}</div>
				<!-- Address -->
				<a
					v-if="
						operation.type === 'transaction' &&
						operation.sender === $store.state.connectedAddress
					"
					rel="noopener"
					target="_blank"
					:href="`https://tzstats.com/${operation.receiver}`"
					>{{ operation.receiver.slice(0, 5) }}...{{
						operation.receiver.slice(-5)
					}}</a
				>
				<a
					v-else-if="
						operation.type === 'transaction' &&
						operation.sender !== $store.state.connectedAddress
					"
					rel="noopener"
					target="_blank"
					:href="`https://tzstats.com/${operation.sender}`"
					>{{ operation.sender.slice(0, 5) }}...{{
						operation.sender.slice(-5)
					}}</a
				>
				<span v-else>&mdash;</span>
				<!-- Hash -->
				<a
					rel="noopener"
					target="_blank"
					:href="`https://tzstats.com/${operation.hash}`"
					>{{ operation.hash.slice(0, 5) }}...{{ operation.hash.slice(-5) }}</a
				>
			</div>
		</section>
		<footer class="is-size-7">
			<div class="left">
				<div class="node-status-container">
					{{ $t("rpc_node_status") }}:&nbsp;
					<span
						:class="{
							'node-status': true,
							'node-status--online': rpcNodeOnline === true,
							'node-status--offline': rpcNodeOnline === false,
						}"
						>{{ rpcNodeOnline ? $tUpper("online") : $tUpper("offline") }}</span
					>
				</div>
				<div class="node-status-container">
					{{ $t("tzstats_api_status") }}:&nbsp;
					<span
						:class="{
							'node-status': true,
							'node-status--online': tzStatsApiOnline === true,
							'node-status--offline': tzStatsApiOnline === false,
						}"
						>{{
							tzStatsApiOnline ? $tUpper("online") : $tUpper("offline")
						}}</span
					>
				</div>
			</div>
			<div class="right">
				<a
					href="https://github.com/lousando/briskett"
					rel="noopener"
					target="_blank"
				>
					<svg
						aria-hidden="true"
						focusable="false"
						data-prefix="fab"
						data-icon="github"
						role="img"
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 496 512"
						width="20"
					>
						<path
							fill="black"
							d="M165.9 397.4c0 2-2.3 3.6-5.2 3.6-3.3.3-5.6-1.3-5.6-3.6 0-2 2.3-3.6 5.2-3.6 3-.3 5.6 1.3 5.6 3.6zm-31.1-4.5c-.7 2 1.3 4.3 4.3 4.9 2.6 1 5.6 0 6.2-2s-1.3-4.3-4.3-5.2c-2.6-.7-5.5.3-6.2 2.3zm44.2-1.7c-2.9.7-4.9 2.6-4.6 4.9.3 2 2.9 3.3 5.9 2.6 2.9-.7 4.9-2.6 4.6-4.6-.3-1.9-3-3.2-5.9-2.9zM244.8 8C106.1 8 0 113.3 0 252c0 110.9 69.8 205.8 169.5 239.2 12.8 2.3 17.3-5.6 17.3-12.1 0-6.2-.3-40.4-.3-61.4 0 0-70 15-84.7-29.8 0 0-11.4-29.1-27.8-36.6 0 0-22.9-15.7 1.6-15.4 0 0 24.9 2 38.6 25.8 21.9 38.6 58.6 27.5 72.9 20.9 2.3-16 8.8-27.1 16-33.7-55.9-6.2-112.3-14.3-112.3-110.5 0-27.5 7.6-41.3 23.6-58.9-2.6-6.5-11.1-33.3 2.6-67.9 20.9-6.5 69 27 69 27 20-5.6 41.5-8.5 62.8-8.5s42.8 2.9 62.8 8.5c0 0 48.1-33.6 69-27 13.7 34.7 5.2 61.4 2.6 67.9 16 17.7 25.8 31.5 25.8 58.9 0 96.5-58.9 104.2-114.8 110.5 9.2 7.9 17 22.9 17 46.4 0 33.7-.3 75.4-.3 83.6 0 6.5 4.6 14.4 17.3 12.1C428.2 457.8 496 362.9 496 252 496 113.3 383.5 8 244.8 8zM97.2 352.9c-1.3 1-1 3.3.7 5.2 1.6 1.6 3.9 2.3 5.2 1 1.3-1 1-3.3-.7-5.2-1.6-1.6-3.9-2.3-5.2-1zm-10.8-8.1c-.7 1.3.3 2.9 2.3 3.9 1.6 1 3.6.7 4.3-.7.7-1.3-.3-2.9-2.3-3.9-2-.6-3.6-.3-4.3.7zm32.4 35.6c-1.6 1.3-1 4.3 1.3 6.2 2.3 2.3 5.2 2.6 6.5 1 1.3-1.3.7-4.3-1.3-6.2-2.2-2.3-5.2-2.6-6.5-1zm-11.4-14.7c-1.6 1-1.6 3.6 0 5.9 1.6 2.3 4.3 3.3 5.6 2.3 1.6-1.3 1.6-3.9 0-6.2-1.4-2.3-4-3.3-5.6-2z"
						></path>
					</svg>
				</a>
				<div>{{ $tCap("version") }}: {{ version }}</div>
				<div>
					<a
						href="https://ssameerhrizvi.artstation.com/"
						rel="noopener"
						target="_blank"
					>
						{{ $t("logo_attribution") }}
					</a>
				</div>
				<div>
					<a href="https://fontawesome.com/" rel="noopener" target="_blank">
						{{ $t("font_awesome_attribution") }}
					</a>
				</div>
				<div>
					{{ $tCap("donations") }}: tz1Sx6bVpeRCQGzmHMCH63AYJ8NcJGswi2sa
				</div>
			</div>
		</footer>
	</main>
</template>

<script>
import Vue from "vue";
import TrezorConnect, { DEVICE, DEVICE_EVENT } from "@trezor/connect-web";
import localeEmoji from "locale-emoji";
import { version } from "../package.json";
import TezosAddressPaths from "../assets/js/tezosAddressPaths";

export default Vue.extend({
	data() {
		return {
			version,
			trezorConnected: false,
			rpcNodeOnline: false,
			tzStatsApiOnline: false,
		};
	},
	head() {
		const i18nHead = this.$nuxtI18nHead({ addSeoAttributes: true });
		return {
			meta: [
				{
					hid: "description",
					name: "description",
					content: this.$t("main_description"),
				},
				...i18nHead.meta,
			],
		};
	},
	mounted() {
		// start from index each time
		this.$nuxt.$router.push("/");

		TrezorConnect.on(DEVICE_EVENT, (event) => {
			if (event.type === DEVICE.CONNECT) {
				this.trezorConnected = true;
			} else if (event.type === DEVICE.DISCONNECT) {
				this.trezorConnected = false;
			}
		});

		const checkNodeStatuses = () => {
			fetch(
				`${process.env.NUXT_ENV_TAQUITO_RPC_URL}/chains/main/blocks/head/hash`
			)
				.then((r) => {
					if (r.ok) {
						this.rpcNodeOnline = true;
					} else {
						this.rpcNodeOnline = false;
					}
				})
				.catch(() => {
					this.rpcNodeOnline = false;
				});

			fetch(`${process.env.NUXT_ENV_TZSTATS_URL}/explorer/tip`, {
				headers: {
					"X-API-Key": process.env.NUXT_ENV_TZPRO_API_KEY || ""
				}
			})
				.then((r) => {
					if (r.ok) {
						this.tzStatsApiOnline = true;
					} else {
						this.tzStatsApiOnline = false;
					}
				})
				.catch(() => {
					this.tzStatsApiOnline = false;
				});
		};

		checkNodeStatuses();

		setInterval(checkNodeStatuses, 1000 * 60 /* check once every minute */);

		setInterval(() => {
			if (this.$store.state.connectedAddress) {
				this.$store.dispatch("loadConnectedAccountData");
			}
		}, 1000 * 30 /* check once every 30 seconds */);
	},
	methods: {
		localeEmoji,
		onLocaleChange({ target }) {
			window.location.href = `https://${target.value}`;
		},
		getCurrentAddressNum() {
			return (
				TezosAddressPaths.indexOf(this.$store.state.connectedAccountPath) + 1
			);
		},
	},
});
</script>

<style lang="scss">
@import "~bulma";

:root {
	--primary-color: #bf545e;
	--secondary-color: #bf545e;
	--tertiary-color: #7d5032;
	--section-background-color: #ccb495;
	--background-color: #ffded4;
	--link-color: #99404b;
	--border-radius: 20px;
}

html {
	background-color: var(--background-color);
	overflow-wrap: break-word;
}

@keyframes fade-in {
	from {
		opacity: 0;
	}
}

.transition-main-enter-active {
	animation: fade-in 500ms ease-out;
}

main {
	display: grid;
	grid-template-rows: 1fr auto auto 1fr;
}

header,
section {
	text-align: center;
	width: min(800px, 80vw);
	margin: 30px auto 0 auto;
}

.trezor-status-container {
	position: fixed;
	top: 10px;
	left: 10px;
	background-color: var(--background-color);
	z-index: 10;

	.trezor-status {
		&--connected {
			color: $green;
		}

		&--disconnected {
			color: $red;
		}
	}
}

.address-number-container {
	position: fixed;
	top: 40px;
	left: 10px;
	background-color: var(--background-color);
	z-index: 10;

	.address-number {
		font-weight: bold;
		font-size: 1.1rem;
	}
}

.help-translate {
	position: absolute;
	top: 5px;
	right: 10px;
	background-color: var(--background-color);
	z-index: 10;
}

.language-switcher {
	position: absolute;
	top: 30px;
	right: 10px;
	background-color: white;
	z-index: 10;
}

.logo-container {
	display: inline-flex;
	align-items: flex-end;

	img {
		width: min(380px, 50vw);
	}

	.beta-text {
		color: var(--tertiary-color);
		font-family: monospace;
		font-weight: bold;
		font-size: 2rem;
	}
}

.balance::after {
	content: "ꜩ";
}

section {
	margin: 30px auto;
	padding: 40px 50px 80px 50px;
	border-radius: var(--border-radius);
	color: var(--tertiary-color);
	background-color: var(--section-background-color);
	box-shadow: 5px 5px 0 var(--tertiary-color);
	overflow-x: auto;
}

.operations {
	.header {
		border-bottom: 1px solid $grey-light;
	}

	.operation {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 2%;
		margin-bottom: 10px;

		> a {
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
		}

		.amount::after {
			content: "ꜩ";
		}

		.reward,
		.receive,
		.send {
			&::before {
				content: "";
				display: inline-block;
				mask-image: url("../assets/icons/arrow-right.svg");
				mask-repeat: no-repeat;
				mask-size: 1rem;
				width: 1rem;
				height: 1rem;
				margin-right: 0.5rem;
			}
		}

		.reward::before {
			background-color: $cyan;
			transform: rotate(45deg);
		}

		.receive::before {
			background-color: $green;
			transform: rotate(45deg);
		}

		.send::before {
			background-color: $red;
			transform: rotate(-45deg);
		}

		.reveal {
			cursor: help;

			svg {
				width: 1rem;
				height: 1rem;
			}
		}
	}
}

a {
	color: var(--link-color);
}

a[target="_blank"] {
	path {
		fill: var(--link-color);
	}

	svg:hover {
		path {
			fill: $grey-darker;
		}
	}

	&:hover {
		&::after {
			background-color: $grey-darker;
		}
	}

	&::after {
		content: "";
		display: inline-block;
		transform: translateY(-0.5px);
		mask-image: url("/icons/external-link-alt-solid.svg");
		mask-repeat: no-repeat;
		mask-position: center;
		mask-size: auto;
		width: 0.7em;
		height: 0.7em;
		margin-left: 2px;
		background-color: var(--link-color);
	}
}

@keyframes notification-entrance {
	from {
		opacity: 0;
		transform: scaleY(0);
	}
	to {
		opacity: 1;
		transform: scaleY(1);
	}
}

.notification {
	animation: notification-entrance 300ms linear;

	a {
		white-space: nowrap;
	}
}

.notification:not(.is-success, .is-danger) {
	background-color: transparent;
	border: 2px solid var(--primary-color);
	border-radius: calc(var(--border-radius) / 2);
}

@keyframes drop-shadow {
	0% {
		box-shadow: 0 0 0 0 black;
	}
	100% {
		box-shadow: 2px 2px 5px black;
	}
}

.button.is-primary {
	background-color: var(--primary-color);
	color: white;
}

.button.is-primary:hover {
	background-color: var(--primary-color);
	animation: drop-shadow 300ms cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

.button.is-primary:focus {
	background-color: var(--primary-color);
}

.button.is-primary.is-loading {
	background-color: var(--primary-color);
}

a.nuxt-link-active {
	border-bottom: 3px solid var(--tertiary-color);
}

.tabs a:hover {
	border-bottom: 3px solid var(--secondary-color);
}

input.input[type="text"]:focus,
input.input[type="number"]:focus {
	border-color: var(--secondary-color);
	box-shadow: none;
}

form {
	margin-bottom: 10px;
}

@keyframes expand {
	0% {
		opacity: 0;
		transform: translatey(-15px);
	}
	100% {
		opacity: 1;
		transform: translatey(0);
	}
}

details[open] p {
	animation: expand 200ms linear;
}

summary {
	margin-top: 10px;
	cursor: pointer;
	user-select: none;
}

footer {
	display: flex;
	justify-content: space-between;

	@media (max-width: $tablet) {
		flex-direction: column;
		gap: 20px;
	}

	.node-status-container {
		font-size: 1rem;

		.node-status {
			&--online {
				color: $green;
			}

			&--offline {
				color: $red;
			}
		}
	}

	.left {
		text-align: left;
		margin-left: 20px;

		@media (max-width: $tablet) {
			text-align: center;
			margin: 0;
		}
	}

	.right {
		text-align: right;
		margin-right: 20px;

		@media (max-width: $tablet) {
			text-align: center;
			margin: 0;
		}
	}
}
</style>
