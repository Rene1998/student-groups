<template>
	<b-modal ref="modal-invite-students" hide-title size='lg' ok-only>
		<template #modal-header="{ close }">
			<h5>Pozvať do triedy</h5>
			<svg-cross-icon class="cross text-right" @click="close()"/>
		</template>
		<div class="m-5">
			<p class="mb-5">Skopirujte odkaz nizsie a nazdielajte ho vasim studentom.
				<br> Pomocou neho studenti budu moct poziadat o pridanie do triedy.
			</p>
			<p class="p-above-input">Linka na <b>pozvanie do triedy</b></p>
			<input ref="inv-input" v-model="this.invLink" readonly/>
		</div>
		<template #modal-footer>
			<b-button class="modal-btn" variant="primary" @click="copyLinkToClipBoard()">Skopírovať</b-button>
		</template>
	</b-modal>
</template>

<script>
export default {
	components: {
		'svg-cross-icon': () => import('@/plugins/appzmudri/_theme/icon/cross.svg?inline'),
	},

	data() {
		return {
			invLink: ''
		}
	},

	methods: {
		showModal(invLink) {
			this.invLink = invLink
			this.$nextTick(() => {
				this.$refs['modal-invite-students'].show()
			})
		},
		copyLinkToClipBoard() {
			this.$refs['inv-input'].select()
			document.execCommand('copy')
		}
	}
}
</script>

<style lang="scss" scoped>
	svg {
		cursor: pointer;
	}

	input {
		border: none;
		border-bottom: 1px solid #CADEEA;
		width: 100%;
		&::placeholder {
			color: rgba(156, 161, 174, 0.4) !important;
			font-family: Poppins;
			font-style: normal;
			font-weight: bold;
			font-size: 18px;
			line-height: 27px;
		}
		&:focus {
			outline: 0;
		}
		&:read-only {
			background-color: transparent;
		}
	}

	/deep/ .modal-content {
		top: 150px;
	}

	/deep/.modal {
		.modal-dialog {
			.modal-content {
				.modal-header {
					border-bottom: 1px solid #E2EDF3 !important;
					width: 90%;
					margin: auto;
					button {
						color:#2B2E4A;
					}
					border-bottom: 0 none;
					.modal-title {
						font-family: 'Boing' !important;
						font-size: 24px;
						line-height: 29px;
					}
				}
				button {
					color: white !important;
				}
				.p-above-input {
					color: #2B2E4A;
					font-size: 14px;
					line-height: 21px;
				}
				p {
					font-style: normal;
					font-weight: normal;
					font-size: 18px;
				}
				b {
					font-weight: 700;
				}
				input {
					color: #2B2E4A;
					font-weight: 700;
				}
				border-style: solid;
				border-width: 1px;
				border-top-left-radius: 10px;
				border-top-right-radius: 10px;
				border-image: linear-gradient(90.01deg, #0064FF 60.75%, #FB2564 79.89%) 0 0 100% 0/0 0 4.53px 0 stretch;
			}
			.modal-footer {
				button {
					width: 221px;
					height: 54px;
				}
				border-top: 0 none;
			}
		}
	}
</style>