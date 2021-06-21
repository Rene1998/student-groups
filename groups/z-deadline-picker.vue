<template>
	<div class="d-flex flex-row light-border w-100 m-height">
		<div class="mr-5">
			<slot class="mb-1" name="calendar-title"></slot>
			<b-form-datepicker selected-variant="pink-light" no-flip today-variant="pink-light" locale="sk" nav-button-variant="blue-dark" v-model="date"
				v-bind="label || {}" :min="min" :hide-header="true" class="calendar">
				<template v-slot:button-content>
					<svg-calendar-icon />
				</template>
			</b-form-datepicker>
		</div>
		<div  class="ml-5">
			<slot name="clock-title"></slot>
			<b-form-group ref="modal-clock-group">
				<b-form-timepicker ref="modal-clock" locale="sk" v-model="time" hide-header no-close-button label-no-time-selected="Vyberte čas">
					<template v-slot:button-content required>
						<svg-clock-icon />
					</template>
				</b-form-timepicker>
			</b-form-group>
		</div>
	</div>
</template>

<script>
export default {
	components: {
		'svg-calendar-icon': () => import('@/plugins/appzmudri/_theme/icon/calendar.svg?inline'),
		'svg-clock-icon': () => import('@/plugins/appzmudri/_theme/icon/clock.svg?inline'),
	},

	watch: {
		date: {
			immediate: true,
			handler(v) {
				this.$emit('date', v)
			}
		},
		time: {
			immediate: true,
			handler(v) {
				this.$emit('time', v)
			}
		}
	},

	data() {
		const now = new Date()
		const todayDate  = new Date(now.getFullYear(), now.getMonth(), now.getDate())
		return {
			min: todayDate,
			date: '',
			time: '08:00:00',
			label: {
				labelPrevDecade: 'Minulé desaťročie',
				labelPrevYear: 'Minulý rok',
				labelPrevMonth: 'Minulý mesiac',
				labelCurrentMonth: 'Tento mesiac',
				labelNextMonth: 'Ďalší mesiac',
				labelNextYear: 'Ďalší rok',
				labelNextDecade: 'Ďalšie desaťročie',
				labelToday: 'Dnes',
				labelSelected: 'Vybraný dátum',
				labelNoDateSelected: 'Vyberte dátum',
				labelCalendar: 'Kalendár',
				labelNav: 'Navigácia v kalendári',
				labelHelp: ''
			}
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

	.calendar, /deep/ .custom-select, /deep/ .b-form-timepicker {
		border: none !important;
		background: none;
		&:active {
			border: none !important;
		}
		//border-bottom: 1px solid #CADEEA !important;
	}

	/deep/ .btn {
		padding: 0;
	}

	.m-height {
		max-height: 70px;
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