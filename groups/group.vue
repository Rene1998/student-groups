<template>
	<b-container>
		<b-row>
			<b-col>
				<div class="d-flex align-items-center ml-3">
					<div class="z-icon-wrapper border cursor-pointer" @click="$router.push('/triedy')">
						<svg-arrow-left-icon />
					</div>
					<div v-if="group">
						<h1 class="ml-4 mb-0" v-if="!editMode">{{group.name}}</h1>
						<input class="ml-4 mb-0" v-if="group.name && editMode"
							:style="'width:' + (group.name.length + 1) + 'ch;'"
							v-model="editedGroup.name">
						<h1 ref="second_name-input" class="ml-2 mb-0" v-if="!editMode">{{group.second_name}}</h1>
						<input v-if="group.second_name && editMode"
							:style="'width:' + (group.second_name.length + 1) + 'ch;'"
							v-model="editedGroup.second_name">
						<b-button class="edit-btn ml-2 pt-1 pb-1 pr-2 pl-2" variant="outline-secondary" v-if="!editMode && isAdmin"
							@click="editMode = true">Upraviť </b-button>
						<b-button class="edit-btn mr-2 pt-1 pb-1 pr-2 pl-2" variant="outline-secondary" v-if="editMode"
							@click="editGroup(), editMode = false">Uložiť </b-button>
						<b-button class="edit-btn pt-1 pb-1 pr-2 pl-2" variant="outline-secondary" v-if="editMode"
							@click="editMode = false, editedGroup.name = group.name, editedGroup.second_name = group.second_name">Zrušiť </b-button>
					</div>
				</div>
			</b-col>
		</b-row>
		<b-row>
			<b-col>
				<div class="text-right pt-3">
					<b-button type="button" variant="outline-primary" class="button-low" v-if="tabIndex == 0 && isAdmin"
						@click="createInvitationLink()" ><svg-share-icon/> Pozvať do triedy</b-button>
					<b-button type="button" variant="outline-primary" class="button-low" v-else-if="tabIndex == 1 && isAdmin"
						@click="$refs['create-task-modal'].showModal()">+ Nová úloha</b-button>
				</div>
				<div class="d-flex flex-row-reverse outer-div-of-yellow-box" v-if="!isAdmin">
					<div class="yellow-box">
						<div class="d-flex box-content mx-auto mt-4">
							<svg-light-bulb-icon style="margin-top: -5px" class="mr-2"/>
							<p class="fw-bold modal-box-text m-auto">Na splnenie úlohy máš len jeden pokus.</p>
						</div>
					</div>
				</div>

				<b-tabs v-model="tabIndex">
					<b-col>
						<b-tab title="Zoznam študentov" active class="mb-5" v-if="isAdmin">
							<div class="d-flex align-items-center mt-4 mb-4">
								<div class="z-icon-wrapper border">
									<svg-gc-invited-student-icon/>
								</div>
								<div>
									<h1 class="ml-4 mb-0" v-if="invitedMembers">Žiadosti o pridanie ({{invitedMembers.length}})</h1>
								</div>
							</div>
							<z-group-table v-if="invitedMembers" class="invitational-table"
								:tableItems="invitedMembers" :tableFields="invitedMemberFields" :tableHeader="'d-none'"
								@studentConfirmation="studentConfirmation" @studentToRemove="$refs['remove-modal'].showModal($event)"/>
							<div class="d-flex align-items-center mt-4 mb-4">
								<div class="z-icon-wrapper border">
									<svg-gc-active-student-icon/>
								</div>
								<div>
									<h1 class="ml-4 mb-0" v-if="activeMembers">Zoznam študentov ({{activeMembers.length}})</h1>
								</div>
							</div>
							<z-group-table v-if="activeMembers"
								:tableItems="activeMembers" :tableFields="activeMembersfields"
								@studentToRemove="$refs['remove-modal'].showModal($event)"/>
							<div class="d-flex align-items-center mt-4 mb-4">
								<div class="z-icon-wrapper border">
									<svg-gc-removed-student-icon/>
								</div>
								<div>
									<h1 class="ml-4 mb-0" v-if="deletedMembers">Odstránení študenti ({{deletedMembers.length}})</h1>
								</div>
							</div>
							<z-group-table v-if="deletedMembers"
								:tableItems="deletedMembers" :tableFields="deletedMembersfields" :tableHeader="'d-none'"
								@itemToReactivate="reactivate" @studentToRemove="$refs['remove-modal'].showModal($event)"/>
						</b-tab>
						<b-tab title="Úlohy">
							<a-tasks ref="tasks"/>
						</b-tab>
					</b-col>
				</b-tabs>
			</b-col>
		</b-row>
		<b-row>
		</b-row>
		<a-invitational-modal ref="invitational-modal" v-if="group"/>
		<a-remove-modal ref="remove-modal" v-if="group"
			@remove="removeStudentFromGroup" :group="group"/>
		<a-create-task-modal ref="create-task-modal"
			:courses="courses" @createTask="createTask"/>
	</b-container>
</template>


<script>
import sortBy from 'lodash/sortBy'
import { mapGetters } from 'vuex'
import wAxios from '@/plugins/w/axios'

const api = {
	admin: {
		getGroup: (gid)								=> 			wAxios.get_auth_data	 (`v1/group-admin/groups/${gid}`),
		acceptMember: (gid, mid)			=> 			wAxios.post_auth_data	(`v1/group-admin/groups/${gid}/members/${mid}/accept`),
		rejectMember: (gid, mid)			  => 		  wAxios.post_auth_data	  (`v1/group-admin/groups/${gid}/members/${mid}/reject`),
		reactivateMember: (gid, mid)	   =>		   wAxios.post_auth_data   (`v1/group-admin/groups/${gid}/members/${mid}/reactivate`),
		removeStudentFromGroup: (gid, mid)	=> wAxios.delete_auth_data (`v1/group-admin/groups/${gid}/members/${mid}`),
		editGroup: (gid, group)				=> 			   wAxios.patch_auth_data  (`v1/group-admin/groups/${gid}`, group),
		createTask: (gid, task)				=> 				  wAxios.post_auth_data    (`v1/group-admin/groups/${gid}/tasks`, task),
		// fetchHash: (gid, eDate) => wAxios.post_auth_data(`v1/group-admin/groups/${gid}/mass-invite`, eDate),
	},
	member: {
		getGroup: (gid) => wAxios.get_auth_data(`v1/group-member/groups/${gid}`),
	},
	getCourses: () => wAxios.get_data('v1/courses'),
}


export default {
	layout: 'profile',

	components: {
		'a-tasks': () => import('@/plugins/lib@profile/tasks/a-tasks.vue'),
		'z-group-table': () => import('@/plugins/lib@profile/groups/z-group-table.vue'),
		'a-remove-modal': () => import('@/plugins/lib@profile/groups/modals/a-remove-student-modal.vue'),
		'a-create-task-modal': () => import('@/plugins/lib@profile/groups/modals/a-create-task-modal.vue'),
		'a-invitational-modal': () => import('@/plugins/lib@profile/groups/modals/a-student-invitational-modal.vue'),
		'svg-arrow-left-icon': () => import('@/plugins/appzmudri/_theme/icon/arrow-left.svg?inline'),
		'svg-share-icon': () => import('@/plugins/appzmudri/_theme/icon/share.svg?inline'),
		'svg-light-bulb-icon': () => import('@/plugins/appzmudri/_theme/icon/light-bulb.svg?inline'),
		'svg-gc-active-student-icon': () => import('@/plugins/appzmudri/_theme/icon/gc-active-student.svg?inline'),
		'svg-gc-invited-student-icon': () => import('@/plugins/appzmudri/_theme/icon/gc-invited-student.svg?inline'),
		'svg-gc-removed-student-icon': () => import('@/plugins/appzmudri/_theme/icon/gc-removed-student.svg?inline'),
	},

	data() {
		return {
			activeMembersfields: [
				{key: 'student_name', label: 'Meno', class: 'name'},
				{key: 'email', label: 'E-mail', class: 'email'},
				{key: 'active_tasks', label: 'Počet vypracovaných zadaní'},
				{key: 'watch_member', label: '', class:'align-end'},
				{key: 'remove', label: '', class: 'cancel'}
			],
			invitedMemberFields: [
				{key: 'student_name', label: 'Meno', class: 'name white'},
				{key: 'email', label: 'E-mail', class: 'email white'},
				{key: 'accept', label: ''},

			],
			deletedMembersfields: [
				{key: 'student_name', label: '', class: 'name low-opacity'},
				{key: 'email', label: '', class: 'email low-opacity'},
				{key: 'is_removed', label: '', class: 'align-end'}
			],
			group: null,
			courses: null,
			activeMembers: null,
			invitedMembers: null,
			deletedMembers: null,
			invLink: null,
			tabIndex: 0,
			editMode: false,
			editedGroup: {
				name: '',
				second_name: ''
			}
		}
	},

	async created() {
		await this._loadGroup()
		await this._loadCourses()
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
		async editGroup() {
			this.$wToast.clear()
			try {
				await api.admin.editGroup(this.$route.params.id, this.editedGroup)
				this.$wToast.success()
				this.$nextTick(async () => {
					await this._loadGroup()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async reactivate(id) {
			this.$wToast.clear()
			try {
				await api.admin.reactivateMember(this.$route.params.id, id)
				this.$wToast.success()
				this.$nextTick(async () => {
					await this._loadGroup()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async removeStudentFromGroup(itemToRemove) {
			this.$wToast.clear()
			try {
				await api.admin.removeStudentFromGroup(this.$route.params.id, itemToRemove.id)
				this.$wToast.success()
				this.$nextTick(async () => {
					await this._loadGroup()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async createInvitationLink(){//http://www.zmudri.sk/triedy?hash=${this.group.invite_hash}
			this.$wToast.clear()
			try {
				// const r = await api.admin.fetchHash(this.$route.params.id, {
				// 	"expires_at": moment().add(7, 'days').format('YYYY-MM-DD HH:MM:SS').toString()
				// })
				const invitational_link = `${window.location.protocol}//${window.location.host}/triedy?hash=${this.group.invite_hash}`
				this.$refs['invitational-modal'].showModal(invitational_link)
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async createTask(task) {
			this.$wToast.clear()
			try {
				await api.admin.createTask(this.$route.params.id, task)
				this.$nextTick(async () => {
					await this.$refs['tasks']._loadTasks()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async studentConfirmation(e) {
			this.$wToast.clear()
			try {
				e.confirmed ? await api.admin.acceptMember(this.$route.params.id, e.id) : await api.admin.rejectMember(this.$route.params.id, e.id)
				this.$nextTick(async () => {
					await this._loadGroup()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async _loadGroup() {
			this.group = this.isAdmin ? await api.admin.getGroup(this.$route.params.id) : await api.member.getGroup(this.$route.params.id)
			this.editedGroup= {name: this.group.name, second_name: this.group.second_name}
			this.invitedMembers = sortBy(this.group.members.filter(m => !m.is_active && m.is_invited && !m.is_removed), e => e.user.name)
			this.activeMembers = sortBy(this.group.members.filter(m => m.is_active && m.is_invited && !m.is_removed), e => e.user.name)
			this.deletedMembers = sortBy(this.group.members.filter(m => m.is_removed), e => e.user.name)
		},
		async _loadCourses() {
			this.courses = await api.getCourses()
		}
	},

}
</script>

 <style lang="scss" scoped>
	.button-low{
		margin-bottom: -90px;
	}
	.outer-div-of-yellow-box {
		position: relative;
		bottom: -45px;
		.yellow-box{
			width: 400px;
			height: 94px;
			background: #FFF5D1;
			border: 1px solid #E6D8A7;
			border-radius: 4px;

			.box-content {
				width: 95%;
			}

			.modal-box-text{
				color: #CAB979;
				font-size: 14px;
			}
		}
	}

	.white {
		color: white !important;
	}

	.invitational-table {
		background-color: #0064FF;
	}

	button {
		color: #86B2CD !important;
		border-color: #86B2CD !important;
		font-size: small;
		font-weight: bold;
		&:hover {
			background-color: #FB2564;
			color: #FFFFFF !important;
			border-color: #FB2564 !important;
		}
	}

	.a-icon-wrapper {
		svg {
			max-height: 100% !important;
		}
	}

	.cursor-pointer {
		cursor: pointer;
	}

	h1 {
		font-size: 24px;
		display: inline-block;
		color:#2B2E4A;
	}

	p {
		font-size: 18px;
		line-height: 21px;
		//color: #000;
	}

	input {
		border: none;
		color:#2B2E4A;
		font-size: 24px;
		font-weight: 900;
		outline: none !important;
	}

	input.reactive {
		width: 5ch;
	}

	.edit-btn {
		font-family: Poppins;
		font-style: normal !important;
		font-weight: 500 !important;
		font-size: 14px !important;
		line-height: 21px !important;
		text-align: center !important;

		color: #2B2E4A !important;
		background: transparent !important;

		&:hover {
			color:#2B2E4A !important;
		}

		border: 1px solid #333333 !important;
		box-sizing: border-box !important;
		border-radius: 4px !important;
	}
</style>