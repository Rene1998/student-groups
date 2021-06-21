<template>
	<div>
		<b-row>
			<b-col>
				<div class="a-profile-subpage-title -groups w-background-orange-light">
					<div>
						<h1 class="teacher-color">Moje triedy</h1>
						<p class="w-color-black">Zoznam mojich tried</p>
					</div>
					<div class="z-icon-wrapper -large">
						<svg-group-icon class="teacher-color" />
					</div>
				</div>
			</b-col>
		</b-row>
		<b-row>
			<b-col>
				<div class="text-right" :class="{'mb60': isAdmin}">
					<b-button v-if="isAdmin" class="mt-4" type="button" pill size="sm" variant="outline-primary mb-0"
						@click="$refs['create-group-modal'].showModal()">+ Vytvoriť triedu</b-button>
				</div>
				<b-tabs>
					<b-col>
						<b-tab title="Moje Triedy" active>
							<z-group-table class="awaiting-confirmation-groups mt-5" v-if="awaitingConfirmationGroups.length > 0 && !isAdmin"
								:tableItems="awaitingConfirmationGroups" :tableFields="studentUnconfirmedFields" :tableHeader="'d-none'"
								@groupToLeave="$refs['remove-group-modal'].showModal($event)"/>
							<z-group-table v-if="groups"
								:tableItems="groups" :tableFields="isAdmin ? adminFields : studentActiveFields"
								@groupToLeave="$refs['remove-group-modal'].showModal($event)" @groupToRemove="$refs['remove-group-modal'].showModal($event)"/>
							<div v-if="!groups && isAdmin" class="d-flex justify-content-center m-5">
								<div class="m-5 text-center" >
									<h5>Zatiaľ ste si nevytvorili žiadnu svoju triedu. </h5>
									<b-button class="mt-3" type="button" pill size="sm" variant="primary mb-0"
										@click="$refs['create-group-modal'].showModal()">+ Vytvorte si ju tu</b-button>
								</div>
							</div>
						</b-tab>
					</b-col>
				</b-tabs>
			</b-col>
		</b-row>
		<b-row>
		</b-row>
		<a-create-group-modal ref="create-group-modal" @createGroup="createGroup"/>
		<a-remove-group-modal v-if="groups"
			:modalConfig="modalConfig" ref="remove-group-modal" @remove="removeGroupOrLeave"/>
		<a-join-group-modal ref="join-group-modal" @refresh="_loadGroups"/>
	</div>
</template>

<script>
import { pullAllWith, isEqual, sortBy } from 'lodash'
import { mapGetters } from 'vuex'
import wAxios from '@/plugins/w/axios'

const api = {
	admin: {
		getGroups: () => wAxios.get_auth_data(`v1/group-admin/groups`),
		createGroup:(group) => wAxios.post_auth_data(`v1/group-admin/groups`, group),
		removeGroup: (gid) => wAxios.delete_auth(`v1/group-admin/groups/${gid}`),
	},
	member: {
		getGroups: () => wAxios.get_auth_data(`v1/group-member/groups`),
		joinGroup: (hash) => wAxios.post_auth_data(`v1/group-member/groups/link-join/${hash}`),
		leaveGroup: (gid) => wAxios.post_auth_data(`v1/group-member/groups/${gid}/leave`),
	}
}

export default {
	layout: 'profile',

	components: {
		'z-group-table':		() => import('@/plugins/lib@profile/groups/z-group-table.vue'),
		'a-remove-group-modal':	() => import('@/plugins/lib@profile/groups/modals/a-remove-group-modal.vue'),
		'a-join-group-modal':	() => import('@/plugins/lib@profile/groups/modals/a-join-group-modal.vue'),
		'a-create-group-modal':	() => import('@/plugins/lib@profile/groups/modals/a-create-group-modal'),
		'svg-group-icon':		() => import('@/plugins/appzmudri/_theme/icon/groups.svg?inline'),
	},

	data() {
		return {
			adminFields: [
				{key: 'name', label: 'Trieda', class: 'subject'},
				{key: 'second_name', label: 'Predmet', class: 'class'},
				{key: 'members_count', label: 'Počet Študentov', class: 'members_count'},
				{key: 'active_tasks', label: 'Počet aktívnych zadaní', class: 'active_task_count'},
				{key: 'space', label: ''},
				{key: 'watch_group', label: ''},
				{key: 'remove', label: '', class: 'cancel'}
			],
			studentActiveFields: [
				{key: 'name', label: 'Trieda', class: 'group'},
				{key: 'second_name', label: 'Predmet', class: 'subject'},
				{key: 'active_tasks', label: 'Počet aktívnych zadaní', class: 'active_task_count'},
				{key: 'watch_group', label: '', class: 'd-flex justify-content-end'},
				{key: 'remove', label: '', class: 'cancel'}
			],
			studentUnconfirmedFields: [
				{key: 'second_name', label: 'Trieda', class: 'group white' },
				{key: 'name', label: 'Predmet', class: 'white'},
				{key: 'space', label: ''},
				{key: 'waiting', label: '', class: 'white'},
				{key: 'remove', label: '', class: 'cancel white'}
			],
			awaitingConfirmationGroups: [],
			groups: null,
			modalConfig: {
				adminHeaderText: 'Naozaj chcete vymazať tento profile?',
				adminBodyText: 'Ste si istý, že chcete odstraniť ',
				studentBodyText: 'Ste si istý, že chcete odísť z ',
				studentHeaderText: 'Naozaj chcete odísť z triedy?'
			}
		}
	},

	watch: {
		'$route.query.hash': {
			immediate: true,
			handler(hash) {
				hash && !this.isAdmin && this._joinGroup(hash)
				hash && this.isAdmin && this.$router.replace( { query: null } )
			},
		},
	},

	computed: {
		...mapGetters('wAuth', [
			'user'
		]),
		isAdmin() {
			return this.user.profile.type == 'teacher'
		}
	},

	async created() {
		await this._loadGroups()
	},

	methods: {
		async removeGroupOrLeave(itemToRemove) {
			this.$wToast.clear()
			try {
				this.isAdmin ? await api.admin.removeGroup(itemToRemove.id) : await api.member.leaveGroup(itemToRemove.id)
				this.$wToast.success()
				this.$nextTick(async () => {
					await this._loadGroups()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async createGroup(group) {
			this.$wToast.clear()
			try {
				await api.admin.createGroup(group)
				this.$wToast.success()
				this.$nextTick(async () => {
					await this._loadGroups()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async _loadGroups() {
			this.groups = this.isAdmin ? await api.admin.getGroups() : await api.member.getGroups()
			this.awaitingConfirmationGroups.length = 0
			this.groups = this.isAdmin ? sortBy(this.groups, e => e.name) : sortBy(this.groups, e => e.group.name)
			this.groups.forEach(e => !e.is_active && e.is_invited && this.awaitingConfirmationGroups.push(e))
			pullAllWith(this.groups, this.awaitingConfirmationGroups, isEqual)
		},
		async _joinGroup(hash) {
			try {

				const r = await api.member.joinGroup(hash)
				setTimeout(() => {
					this.$router.replace({'query': null})
					this.$refs['join-group-modal'].showModal(r.group)
					this.$nextTick(async () => {
						this.$router.replace( { query: null } )
						await this._loadGroups()
					})
				}, 50);
			} catch(e) {
				this.$nextTick(() => {
					this.$router.replace( { query: null } )
				})
				this.$wToast.error(e)
			}
		}
	}
}
</script>

<style lang="scss" scoped>
	.svg-pointer {
		cursor: pointer;
	}

	.mb60 {
		margin-bottom: -60px
	}

	.awaiting-confirmation-groups {
		color: white !important;
		background-color: #0064FF;
		/deep/ .cancel {
			padding-top: 12px;
		}
	}

	.teacher-color {
		//background: #F2994A;
		color: #F2994A;
	}

	h1 {
		color: #2B2E4A;
	}

	button {
		color: #86B2CD !important;
		border-color: #86B2CD !important;
		min-width: 136px;
		font-size: small;
		font-weight: bold;
		&:hover {
			background-color: #FB2564;
			color: #FFFFFF !important;
			border-color: #FB2564 !important;
		}
	}
</style>