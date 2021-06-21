<template>
	<b-modal ref="modal-remove-group" hide-title ok-only ok-title="Zavrieť" size='lg' v-if="this.itemToRemove"
		@ok="$emit('remove', itemToRemove)">
		<template #modal-header="{ close }">
			<h5 v-if="isAdmin"> {{modalConfig.adminHeaderText}}</h5>
			<h5 v-else> {{modalConfig.studentHeaderText}}</h5>
			<svg-cross-icon @click="close()"/>
		</template>
		<div class="m-5">
			<p v-if="isAdmin">{{modalConfig.adminBodyText}}
				<b class="fw-bold">{{itemToRemove.second_name}} {{itemToRemove.name}} </b>
				?
			</p>
			<p v-else>{{modalConfig.studentBodyText}}
				<b class="fw-bold">{{itemToRemove.second_name}} {{itemToRemove.name}} </b>
				?
			</p>
		</div>
		<template #modal-footer="{ ok, cancel }">
			<b-button class="modal-btn" variant="primary" @click="ok()">ÁNO</b-button>
			<b-button class="modal-btn" variant="outline-danger" @click="cancel()">NIE</b-button>
		</template>
	</b-modal>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
	components: {
		'svg-cross-icon': () => import('@/plugins/appzmudri/_theme/icon/cross.svg?inline'),
	},

	props: {
		modalConfig: {
			required: true,
			type: Object,
			adminHeaderText: {
				required: true,
				type: String
			},
			adminBodyText: {
				required: true,
				type: String
			},
			studentBodyText : {
				required: false,
				type: String
			},
			studentHeaderText: {
				required: false,
				type: String
			},
		}
	},

	data() {
		return {
			itemToRemove: null
		}
	},

	computed: {
		...mapGetters('wAuth', [
			'user'
		]),
		isAdmin() {
			return this.user.profile.type == 'teacher'
		}
	},

	methods: {
		showModal(e) {
			'course' in e ? this.itemToRemove = e.course : this.itemToRemove = e
			this.$nextTick(() => {
				this.$refs['modal-remove-group'].show()
			})
		}
	}
}
</script>

<style lang="scss" scoped>
	svg {
		cursor: pointer;
	}

	h4 {
		font-family: Boing !important;
		font-style: normal;
		font-weight: bold;
		font-size: 24px;
		line-height: 29px;
		color: #2B2E4A;
	}

	/deep/ .modal-header {
		border-bottom: 0 none;
	}

	/deep/ .modal-content {
		top: 150px;
	}

	/deep/ .modal-body {
		p {
			font-family: Roboto;
			font-style: normal;
			font-weight: normal;
			font-size: 18px;
			line-height: 21px;
			margin-top: auto !important;
			margin-bottom: auto !important;
			align-items: center;

			color: #000000;
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