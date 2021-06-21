<template>
	<b-modal ref="modal-create-task" hide-title ok-title="Zavrieť" size='lg' ok-only
		@ok="handleOk" @show="resetCourse()">
		<template #modal-header="{ close }">
			<h5 >Zadanie úlohy</h5>
			<svg-cross-icon @click="close()"/>
		</template>
		<b-form ref="task-form" class="m-5" @submit="handleSubmit">
			<div class="light-border w-100 mb-5">
				<p>Vyberte kurz</p>
				<div class="d-flex flex-row" @click="toggleDropdown">
					<b-dropdown ref="dropdown" size="lg" variant="link" toggle-class="text-decoration-none" no-caret menu-class="auto">
						<template #button-content>
							<svg-arrow-facing-down-icon/>
						</template>
						<b-dropdown-item v-for="course in courses" :key="course.id" class="m-width-of-dd-item">
							<div class="d-flex flex-row" @click="form.selectedCourse = course" >
								<img :src="course.image.path" width="90" height="45" class="mr-5">
								<span class="mt-2">{{course.name}}</span>
							</div>
						</b-dropdown-item>
					</b-dropdown>
					<img v-if="form.selectedCourse.image" :src="form.selectedCourse.image.path" width="90" height="45" class="ml-2 mr-3 mb-1">
					<b-form-input style="border: none !important" placeholder="Vyberte si kurz" required readonly
						v-if="form.selectedCourse" v-model="form.selectedCourse.name"/>
				</div>
			</div>
			<z-deadline-picker @date="form.date = $event" @time="form.time = $event">
				<template v-slot:calendar-title>
					<p>Do kedy treba splniť (dátum)</p>
				</template>
				<template v-slot:clock-title>
					<p>Do kedy treba splniť (čas)</p>
				</template>
			</z-deadline-picker>
		</b-form>
		<template #modal-footer="{ ok, cancel }">
			<b-button class="modal-btn" variant="primary" type="submit" @click="ok()">ÁNO</b-button>
			<b-button class="modal-btn" variant="outline-danger" @click="cancel()">NIE</b-button>
		</template>
	</b-modal>
</template>

<script>

export default {
	components: {
		'z-deadline-picker':		() => import('@/plugins/lib@profile/groups/z-deadline-picker.vue'),
		'svg-cross-icon': () => import('@/plugins/appzmudri/_theme/icon/cross.svg?inline'),
		'svg-arrow-facing-down-icon': () => import('@/plugins/appzmudri/_theme/icon/arrow-facing-down.svg?inline'),
	},

	props: {
		courses : {
			required: true,
			type: Array
		}
	},

	data() {
		return {
			form: {
				selectedCourse: {
					name: ''
				},
				date: '',
				time: '',
			},
			showDropdown: false
		}
	},

	methods: {
		showModal() {
			this.$nextTick(() => {
				this.$refs['modal-create-task'].show()
			})
		},
		toggleDropdown() {
			this.showDropdown = !this.showDropdown
			this.showDropdown ? this.$refs.dropdown.show() : this.$refs.dropdown.hide()
		},
		resetCourse() {
			this.form = {
				selectedCourse: {
					name: ''
				},
				date: '',
				time: ''
			}
		},
		handleOk(bvModalEvt) {
			bvModalEvt.preventDefault()
			this.handleSubmit()
		},
		handleSubmit() {
			if(!this.form.selectedCourse.id || !this.form.date || !this.form.time)
				return

			this.$emit('createTask', {
				course_id: this.form.selectedCourse.id,
				deadline: `${this.form.date} ${this.form.time}`
			})
			this.$nextTick(() => {
				this.$refs['modal-create-task'].hide()
			})
		}
	}
}
</script>

<style lang="scss" scoped>
	svg {
		cursor: pointer;
	}

	span {
		color: #2B2E4A;
		font-family: Poppins;
		font-style: normal;
		font-weight: bold;
		font-size: 18px;
		line-height: 27px;
	}

	h4 {
		font-family: Boing !important;
		font-style: normal;
		font-weight: bold;
		font-size: 24px;
		line-height: 29px;
		color: #2B2E4A;
	}

	.m-width-of-dd-item {
		max-width: 675px;
	}

	.calendar, /deep/ .custom-select, /deep/ .b-form-timepicker {
		border: none !important;
		background: none;
		&:active {
			border: none !important;
		}
		//border-bottom: 1px solid #CADEEA !important;
	}

	/deep/ .text-dark, /deep/ .text-truncate, /deep/ .text-center {
		color: #2B2E4A !important;
	}

	/deep/ .text-muted {
		color: rgba(156, 161, 174, 0.4) !important;
		font-family: Poppins;
		font-style: normal;
		font-weight: bold;
		font-size: 18px !important;
		line-height: 27px !important;
	}

	/deep/ .modal-header {
		border-bottom: 0 none;
	}

	/deep/ .light-border {
		border-bottom: 1px solid #CADEEA;
	}

	/deep/ .modal-content {
		top: 150px;
	}

	/deep/ .modal-body {
		p {
			font-family: Roboto;
			font-style: normal;
			font-weight: normal;
			font-size: 14px;
			line-height: 21px;
			margin-top: auto !important;
			margin-bottom: auto !important;
			align-items: center;

			color: #2B2E4A;
		}
		.modal-box {
			align-items: center;
			width: 569px;
			height: 94px;
			left: 617px;
			top: 847px;

			background: #FFF5D1;
			border: 1px solid #E6D8A7;
			box-sizing: border-box;
			border-radius: 4px;
		}
		.modal-box-text {
			font-family: Poppins;
			font-style: normal;
			font-weight: bold;
			font-size: 14px;
			line-height: 18px;

			letter-spacing: -0.05em;

			color: #CAB979;
		}
	}
	input {
		border: none;
		width: 100%;
		color: #2B2E4A;
		font-family: Poppins;
		font-style: normal;
		font-weight: bold;
		font-size: 18px;
		line-height: 27px;
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


	/deep/.modal-content {
		border-style: solid;
		border-width: 1px;
		border-top-left-radius: 10px;
		border-top-right-radius: 10px;
		border-image: linear-gradient(90.01deg, #0064FF 60.75%, #FB2564 79.89%) 0 0 100% 0/0 0 4.53px 0 stretch;
	}

	/deep/.modal-footer {
		border-top: 0 none !important;
		align-items: left;
	}

	.modal-btn {
		width: 221px;
		height: 54px;
		left: 782px;
		top: 767px;
	}

	.box-content {
		margin: auto;
		padding: 20px;
		justify-content: center;
	}

	.box-bottom {
		width: 100%;
		background: #FFF5D1;
		border: 1px solid #E6D8A7;
		box-sizing: border-box;
		align-items: center ;
		border-radius: 4px;
	}
</style>