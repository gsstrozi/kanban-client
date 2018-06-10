<template>
	<div class="ui basic segment">
		<div class="ui sixteen wide celled grid">
			<!-- Members, Doing, Blocked, and Done columns -->
			<div class="ui sixteen wide column">
				<div class="ui sixteen column celled grid">
					<!-- Column headers -->
					<div class="equal height row">
						<div class="four wide center aligned column">
							<div class="ui basic label">
								<i class="users icon"></i> {{_teamLabel}}
								<a class="detail" v-on:click="newMember" :title="_newMemberHint"><i class="teal plus icon"></i></a>
							</div>
						</div>
						<div class="four wide column">
							<div class="ui red ribbon label">{{_todoLabel}}</div>
						</div>
						<div class="four wide column">
							<div class="ui yellow ribbon label">{{_progressLabel}}</div>
						</div>
						<div class="four wide column">
							<div class="ui green ribbon label">{{_doneLabel}}</div>
						</div>
					</div>
					<!-- For each member, fill the taskboard: todo, progress, and done tasks -->
					<div
						class="equal height row">
						<!-- Members container -->
						<div class="four wide column">
							<div class="row members" :key="member.id" v-for="member in getMembers">
								<img v-if="member.avatar === undefined" class="ui avatar image" src="./assets/unsigned.png">
								<img v-if="member.avatar !== undefined" class="ui avatar image" :src="getImage(member.avatar)">
								<div class="ui basic left pointing label">{{member.name}}</div>
							</div>
						</div>
						<!-- Doing cards container -->
						<div class="four wide column todo">
							<task-card
								v-for="t in getTodoTasks"
								:task="t"
								:key="t.id"
				    			:top-content="_taskTopContent"
								@openTask="openTask"
								@confirmDeleteTask="confirmDeleteTask">
							</task-card>
						</div>
						<!-- Blocked cards container -->
						<div class="four wide column progress">
							<task-card
								v-for="t in getInProgressTasks"
								:task="t"
								:key="t.id"
				    			:top-content="_taskTopContent"
								@openTask="openTask"
								@confirmDeleteTask="confirmDeleteTask">
							</task-card>
						</div>
						<!-- Done cards container -->
						<div class="four wide column done">
							<task-card
								v-for="t in getDoneTasks"
								:task="t"
								:key="t.id"
				    			:top-content="_taskTopContent"
								@openTask="openTask"
								@confirmDeleteTask="confirmDeleteTask">
							</task-card>
						</div>
					</div>					
				</div>	    
			</div>
		</div>
		<task-dialog
			:owners="members"
			@updateTask="updateTask"
			@addTag="addTag"
			@deleteTag="deleteTag"
			ref="dialog">
		</task-dialog>
		<confirm-dialog
			@deleteTask="deleteTask"
			ref="confirm">
		</confirm-dialog>
		<member-dialog
			@updateMember="updateMember"
			ref="memberDialog">
		</member-dialog>
	</div>
</template>

<script type="text/javascript">
	import _ from 'lodash'
	import $ from 'jquery'
	import dragula from 'dragula'
	import 'semantic-ui-css/semantic.min'
	import 'semantic-ui-css/semantic.min.css'
	import TaskCard from './components/taskCard.vue'
	import TaskDialog from './components/taskDialog.vue'
	import ConfirmDialog from './components/confirmDialog.vue'
	import MemberDialog from './components/memberDialog.vue'

	export default {
		components: {
			TaskCard,
			TaskDialog,
			ConfirmDialog,
			MemberDialog
		},
		props: {
			tasks: {
				type: Array,
				required: true
			},
			members: {
				type: Array,
				required: true
			},
			options: {
				type: Object,
				required: false
			}
		},
		computed: {
			getMembers () {
				return this.members
			},
			getTodoTasks () {
				const todoTasks = this.tasks.filter((task) => {
					return task.status === 'todo'
				})
				return todoTasks
			},
			getInProgressTasks () {
				const todoTasks = this.tasks.filter((task) => {
					return task.status === 'progress'
				})
				return todoTasks
			},
			getDoneTasks () {
				const todoTasks = this.tasks.filter((task) => {
					return task.status === 'done'
				})
				return todoTasks
			},
			_columnLabels () {
				if (this.options && this.options.columnLabels) {
					return this.options.columnLabels
				} else {
					return ['Equipe', 'A Fazer', 'Em Progresso', 'Finalizado']
				}
			},
			_teamLabel () {
				return this._columnLabels[0]
			},
			_todoLabel () {
				return this._columnLabels[1]
			},
			_progressLabel () {
				return this._columnLabels[2]
			},
			_doneLabel () {
				return this._columnLabels[3]
			},
			_newTaskHint () {
				return this.options && this.options.newTaskHint
					? this.options.newTaskHint
					: 'New Task'
			},
			_newMemberHint () {
				return this.options && this.options.newMemberHint
					? this.options.newMemberHint
					: 'New Member'
			},
			_backlogExtraContent () {
				return this.options && this.options.backlogExtraContent
					? this.options.backlogExtraContent()
					: null
			},
			_backlogTopContent () {
				return this.options && this.options.backlogTopContent
					? this.options.backlogTopContent()
					: null
			},
			_taskTopContent () {
				return this.options && this.options.taskTopContent
					? this.options.taskTopContent()
					: null
			}
		},
		methods: {
			getImage (avatar) {
				return (avatar || './assets/unsigned.png')
			},
			openTask (task) {
				if (!this.options || this.options.defaultTaskDialog) {
					this.$refs.dialog.show(task)
				} else {
					if (task) {
						this.$emit('openTask', task)
					} else {
						this.$emit('newTask')
					}
				}
			},
			confirmDeleteTask (task) {
				if (!this.options || this.options.defaultConfirmDialog) {
					this.$refs.confirm.show(task)
				} else {
					this.$emit('confirmDeleteTask', task)
				}
			},
			deleteTask (task) {
				this.$emit('deleteTask', task)
			},
			getTaskById (id) {
				const backlogTask = _.find(this._backlog, {id: id})
				if (backlogTask) {
					return backlogTask
				}

				return _.find(this.tasks, {id: id})
			},
			updateTask (task) {
				this.$emit('updateTask', task)
			},
			addTag (task, tag) {
				this.$emit('addTag', task, tag)
			},
			deleteTag (task, tag) {
				this.$emit('deleteTag', task, tag)
			},
			newMember () {
				if (!this.options || this.options.defaultMemberDialog) {
					this.$refs.memberDialog.show()
				} else {
					this.$emit('newMember')
				}
			},
			updateMember (member) {
				this.$emit('updateMember', member)
			},
			configDragula () {
				const self = this
				/*
					Disable swipe-history-navigation
					https://superuser.com/questions/840102/how-do-you-disable-swipe-history-navigation
					Go to chrome://flags/#overscroll-history-navigation
					Disable the Overscroll history navigation experiment:
				*/
				var drake = dragula({
					isContainer: function (el) {
						return el.classList.contains('todo') ||
							el.classList.contains('todo') ||
							el.classList.contains('progress') ||
							el.classList.contains('done')
					},
					moves: function (el, target, source, sibling) {
						var owner = $(target).attr('owner')
						var status = $(target).attr('class').split(' ').reverse()[0]

						var sourceOwner = $(source).attr('owner')
						var sourceStatus = $(source).attr('class').split(' ').reverse()[0]

						if (owner !== sourceOwner || status !== sourceStatus) {
							return true
						} else {
							return false
						}
					},
					accepts: function (el, target, source, sibling) {
						var owner = $(target).attr('owner')
						var status = $(target).attr('class').split(' ').reverse()[0]

						var sourceOwner = $(source).attr('owner')
						var sourceStatus = $(source).attr('class').split(' ').reverse()[0]

						if (owner !== sourceOwner || status !== sourceStatus) {
							return true
						} else {
							return false
						}
					}
				})

				drake.on('drop', function (el, target, source, sibling) {
					var id = $(el).attr('id')
					var owner = $(target).attr('owner')
					var status = $(target).attr('class').split(' ').reverse()[0]

					var sourceOwner = $(source).attr('owner')
					var sourceStatus = $(source).attr('class').split(' ').reverse()[0]

					if (owner !== sourceOwner || status !== sourceStatus) {
						var task = _.assign({
							id: null,
							subject: null,
							description: null,
							dueDate: null,
							tags: []
						}, self.getTaskById(id))
						task.status = status
						task.owner = owner

						self.$emit('updateTask', task)
						return true
					} else {
						return false
					}
				})
			}
		},
		mounted () {
			this.configDragula()
		}
	}
</script>

<style type="text/css">
	/* dragula css */
	.gu-mirror {
		position: fixed !important;
		margin: 0 !important;
		z-index: 9999 !important;
		opacity: 0.8;
		-ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=80)";
		filter: alpha(opacity=80);
	}
	.gu-hide {
		display: none !important;
	}
	.gu-unselectable {
		-webkit-user-select: none !important;
		-moz-user-select: none !important;
		-ms-user-select: none !important;
		user-select: none !important;
	}
	.gu-transit {
		opacity: 0.2;
		-ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=20)";
		filter: alpha(opacity=20);
	}
	.members {
		padding-bottom: 10px;
	}
</style>
