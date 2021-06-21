<template>
	<b-modal ref="modal-create-group" hide-title ok-title="Zavrieť" size='lg' ok-only
		@ok="handleOk" @show="resetGroup()">
		<template #modal-header="{ close }">
			<h5>Naozaj chcete vytvoriť túto triedu?</h5>
			<svg-cross-icon @click="close()"/>
		</template>
		<div class="m-5">

			<form @submit="handleSubmit">
				<b-form-group
					invalid-feedback="Emaily su povinne">
					<p class="p-above-input w-color-black">Názov triedy</p>
					<b-form-input
						v-model="form.name"
						class="mb-4"
						placeholder="napr. 1.A, Chrumkáčikovia">
					</b-form-input>
					<p class="p-above-input w-color-black">Predmet</p>
					<b-form-input
						v-model="form.second_name"
						placeholder="napr. Biológia, Matematika">
					</b-form-input>
				</b-form-group>
			</form>
			<div class="yellow-box">
				<div class="d-flex mx-auto box-content mt-3">
					<svg-light-bulb-icon class="mr-5"/>
					<p class="modal-box-text font-weight-bold">Odporúčame aby ste si predmety a triedy pomenovali XY <br> správny sposobom.</p>
				</div>
			</div>
		</div>
		<template #modal-footer="{ ok }">
			<b-button class="modal-btn" variant="primary" @click="ok()">Vytvoriť </b-button>
		</template>
	</b-modal>
</template>

<script>
export default {
	components: {
		'svg-light-bulb-icon': () => import('@/plugins/appzmudri/_theme/icon/light-bulb.svg?inline'),
		'svg-cross-icon': () => import('@/plugins/appzmudri/_theme/icon/cross.svg?inline')
	},

	data() {
		return {
			form: {
				name: '',
				second_name: ''
			}
		}
	},

	methods: {
		showModal() {
			this.$nextTick(() => {
				this.$refs['modal-create-group'].show()
			})
		},
		resetGroup() {
			this.form= {
				name: '',
				second_name: ''
			}
		},
		handleOk(bvModalEvt) {
			bvModalEvt.preventDefault()
			this.handleSubmit()
		},
		handleSubmit() {
			if(!this.form.name || !this.form.second_name)
				return

			this.$emit('createGroup', this.form)

			this.$nextTick(() => {
				this.$refs['modal-create-group'].hide()
			})
		},
	}
}
</script>

<style lang="scss" scoped>
.yellow-box{
	width: 100%;
	height: 94px;
	background: #FFF5D1;
	border: 1px solid #E6D8A7;
	box-sizing: border-box;
	align-items: center ;
	border-radius: 4px;

	.box-content {
		width: 95%;
	}
}

.modal-box-text{
	color: #CAB979;
}
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

	.p-above-input {
		font-size: 14px;
		line-height: 21px;
	}

	/deep/.modal {
		.modal-dialog {
			.modal-content {
				top: 150px;
				.modal-header {
					border-bottom: 1px solid #E2EDF3 !important;
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
				border-top: 0 !important;
			}
		}
	}
</style>