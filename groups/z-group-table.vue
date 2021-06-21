<template>
	<b-table borderless responsive="xl" large fixed hover v-if="tableItems && tableFields"
		:thead-class="tableHeader"  :fields="tableFields"  :items="tableItems" >
		<template #head()="data">
			<span>{{ data.label }}</span>
		</template>
		<template #cell(name)="row">
			<b v-if="row.item.name && !row.item.group">{{ row.item.name }}</b>
			<b v-if="row.item.group">{{ row.item.group.name }}</b>
			<b v-if="row.item.user && !row.item.group">{{row.item.user.name}} {{row.item.user.surname}}</b>
		</template>
		<template #cell(second_name)="row">
			<b v-if="row.item.second_name">{{ row.item.second_name }}</b>
			<b v-if="row.item.group">{{ row.item.group.second_name }}</b>
		</template>
		<template #cell(courseImg)="row">
			<div class="thumbnail m-auto">
				<img :src="`${row.item.course.image.path}`">
			</div>
		</template>
		<template #cell(course)="row">
			<b> {{row.item.course.name}}</b>
		</template>
		<template #cell(student_name)="row">
			<router-link v-if="row.item.is_active && !row.item.is_removed && row.item.user" :class="'student-name'" :to="`${$route.params.id}/${row.item.id}`">
				<b>{{ row.item.user.name }} {{row.item.user.surname}}</b>
			</router-link>
			<b v-else-if="row.item">{{ row.item.user.name }} {{row.item.user.surname}}</b>
		</template>
		<template #cell(email)="row">
			<p>{{row.item.user.email}}</p>
		</template>
		<template #cell(watch_member)="row">
			<router-link v-if="row.item.is_active && row.item.is_invited" :to="`${$route.params.id}/${row.item.id}`" >
				<b-button  variant="outline-primary" >Pozrieť</b-button>
			</router-link>
		</template>
		<template #cell(deadline)="row">
			<p v-if="row.item.deadline" :class="!row.item.is_active && isAdmin && 'white'">{{formatDate(row.item.deadline)}}</p>
			<p v-else>-</p>
		</template>
		<template #cell(created_at)="row">
			<p v-if="row.item.created_at" :class="!row.item.is_active && isAdmin && 'white'">{{formatDate(row.item.created_at)}}</p>
			<p v-else>-</p>
		</template>
		<template #cell(watch_group)="row" size="sm">
			<router-link v-if="isAdmin" :to="`/triedy/${row.item.id}`"><b-button type="button" variant="outline-secondary" size="sm">Pozrieť</b-button></router-link>
			<router-link v-else :to="`/triedy/${row.item.group.id}`"><b-button type="button" variant="outline-secondary" size="sm">Pozrieť</b-button></router-link>
		</template>
		<template #cell(members_count)="row">
			<p>{{row.item.members_count}}</p>
		</template>
		<template #cell(watch_or_reactivate)="row" size="sm">
			<router-link v-if="row.item.is_active" :to="`${$route.params.id}/uloha/${row.item.id}`">
				<b-button type="button" variant="outline-secondary" size="sm">Pozrieť</b-button>
			</router-link>
			<b-button v-else :class="'reactivate-btn'" @click="emitItemToReactivate(row.item.id)">Reaktivovať</b-button>
		</template>
		<template #cell(go_to_course)="row" size="sm">
			<router-link v-if="row.item.course" :to="`/kurz/${row.item.course.slug}`">
				<b-button type="button" variant="outline-secondary" size="sm">Prejsť si kurz</b-button>
			</router-link>
		</template>
		<template #cell(is_removed)="row">
			<b-button variant="outline-primary" @click="emitItemToReactivate(row.item.id)" class="mr-4 mt-1">Reaktivovať</b-button>
		</template>
		<template #cell(active_tasks)="row">
			<p v-if="row.item.active_tasks">{{row.item.active_tasks}}</p>
			<b v-if="row.item.group && !row.item.group.tasks_count">{{row.item.group.active_tasks}}</b>
			<p v-if="row.item.tasks_count">{{row.item.tasks_count.tasks_done_count}} / {{row.item.tasks_count.tasks_count}}</p>
		</template>
		<template #cell(exams_count)="row">
			<b v-if="row.item.response">{{row.item.response.correct_points}} / {{row.item.response.max_points}}</b>
			<p v-else-if="Array.isArray(row.item.responses) && row.item.responses.length">
				{{row.item.responses[0].correct_points}} / {{row.item.responses[0].max_points}}
			</p>
			<p v-else-if="row.item.task_responses && row.item.task_responses.length > 0">
				{{ row.item.task_responses[0].correct_points}} / {{row.item.task_responses[0].max_points}}
			</p>
			<p v-else :class="'red-group'">Nevypracované</p>
		</template>
		<template #cell(waiting)>
			<b>Čaká sa na schválenie...</b>
		</template>
		<template #cell(remove)="row" size="sm">
			<svg-cross-icon v-if="isAdmin" v-b-popover.hover.top="'Odstrániť'" @click="emitItemToRemove(row.item)"/>
			<svg-cross-icon v-else v-b-popover.hover.top="'Odísť'" @click="emitItemToRemove(row.item)"/>
		</template>
		<template #cell(accept)="row">
			<div class="d-flex float-right">
				<b-button @click="$emit('studentConfirmation', {id: row.item.id, confirmed: true})" :class="'accept-btn mr-2'">Schváliť</b-button>
				<b-button @click="$emit('studentConfirmation', {id: row.item.id, confirmed: false})"  :class="'deny-btn'">Zamietnuť</b-button>
			</div>
		</template>
	</b-table>
</template>

<script>
import moment from 'moment'
import { mapGetters } from 'vuex'

export default {
	components: {
		'svg-cross-icon': () => import('@/plugins/appzmudri/_theme/icon/cross.svg?inline'),
		//'svg-bullet-icon': () => import('@/plugins/appzmudri/_theme/icon/bullet.svg?inline')
	},

	props: {
		tableItems: {
			required: true,
			type: Array
		},
		tableFields: {
			required: true,
			type: Array
		},
		tableHeader: {
			required: false,
			type: String,
			default: ''
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
		emitItemToRemove(item) {
			if ('group' in this.tableItems[0])
				this.$emit('groupToLeave', item.group)
			else if ('user' in this.tableItems[0])
				this.$emit('studentToRemove', item)
			else
				this.$emit('groupToRemove', item)
			//'group' in this.tableItems[0] ? this.$emit('groupToRemove', item): this.$emit('groupToLeave', item.group) ?  'response' in this.tableItems[0] ? this.$emit('taskToRemove', item.task.course) : this.$emit('studentToRemove', item)
		},
		emitItemToReactivate(id) {
			this.$emit('itemToReactivate', id)
		},
		formatDate(v) {
			return moment(v).format('L')
		}
	}
}
</script>

<style lang="scss" scoped>
	.thumbnail {
		height: 48px;
		width: 76px;
		border-radius: 4px;
		overflow: hidden;

		img {
			width: 76px;
			object-fit: cover;
		}

	}
	svg {
		cursor: pointer;
	}

	a {
		text-decoration: none;
		color: #86B2CD;
		font-weight: lighter;
		font-family: Poppins;
		font-style: normal;
		font-weight: 700;
		font-size: 14px;
		line-height: 21px;
		&:hover {
			color: white;
		}
	}

	/deep/ table {
		overflow-y: hidden;
		overflow-x: hidden;
		thead {
			tr {
				border-bottom: 1px solid #CADEEA;
				th {
					text-align: center;
					span {
						color: #86B2CD;
						font-weight: lighter;
						font-family: Poppins;
						font-style: normal;
						font-weight: 300;
						font-size: 14px;
						line-height: 21px;
					}
				}
			}
		}
		tbody {
			tr {
				td {
					// overflow: hidden;
					// text-overflow: ellipsis;
					text-align: center;
					align-items: center;
					color: #2B2E4A;
					button {
						color: #86B2CD !important;
						border-color: #86B2CD !important;
						font-size: small;
						font-weight: bold;
						min-width: 136px;
						min-height: 39px;
						padding: 0;
						&:hover {
							background-color: #FB2564;
							color: #FFFFFF !important;
							border-color: #FB2564 !important;
						}
					}
					a {
						text-decoration: none;
						color: #2B2E4A;
						font-weight: bold;
					}
				}
			}
		}
	}

	/deep/ .name {
		width: 150px;
	}

	/deep/ .reactivate-btn {
		width: 136px;
		background-color: #2B2E4A;
		color: white !important;
		border-radius: 100px;
		border-color: #2B2E4A !important;
	}

	/deep/ .blue-group {
		color: #86B2CD !important;
	}

	/deep/ .red-group {
		color: #F64A4A !important;
	}

	/deep/ .white {
		p,b {
			color: white;
		}
	}

	/deep/ .student-name {
		&:hover {
			color: #2B2E4A !important;
		}
	}

	/deep/ .align-start {
		text-align: start;
	}

	/deep/ .low-opacity {
		opacity: 50%;
	}

	/deep/ .align-end {
		text-align: end;
	}

	/deep/ .members_count{
		width: 159px;
	}

	/deep/ .group {
		width: 100px !important;
		max-width: 100px !important;
	}

	/deep/ .email {
		width: 300px;
	}

	/deep/ .subject2 {
		width: 100px !important;
		max-width: 100px !important;
	}

	/deep/ .cancel {
		color: #86B2CD;
		box-sizing: border-box  !important;
		justify-content: start !important;
		width: 30px;
		padding-top: 18px;
	}

	/deep/ .subject {
		width: 200px !important;
	}

	/deep/.active_task_count {
		width: 200px !important;
	}

	/deep/ .accept-btn {
		color: #FFFFFF !important;
		background-color: #2B2E4A ;
		border-color: #2B2E4A !important;
		&:hover {
			background-color: #FB2564;
			color: #FFFFFF !important;
			border-color: #FB2564 !important;
		}
	}

	.deny-btn {
		width: 136px;
		background-color: #0064FF;
		color: #FFFFFF !important;
		border-color: white !important;
	}

	button {
		&:hover {
			background-color: #FB2564;
			color: #FFFFFF !important;
			border-color: #FB2564 !important;
		}
	}

	.violet {
		color: #FB2564
	}

	// ::-webkit-scrollbar {
	// 	display: none;
	// }
</style>