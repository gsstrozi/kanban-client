<template>
	<semantic-kanban
		:tasks="tasks"
		:members="members"
		:options="options"
		@updateTask="updateTask"
		@deleteTask="deleteTask"
		@addTag="addTag"
		@deleteTag="deleteTag"
		@updateMember="updateMember">
	</semantic-kanban>  
</template>

<script>
import SemanticKanban from './semanticKanban'

export default {
	name: 'app',
	components: {
		SemanticKanban
	},
	data () {
		return {
			tasks: [{
				id: 3,
				subject: '[Tacógrafo] - OS0010',
				description: 'Observações',
				status: 'todo',
				owner: 2,
				type: '',
				tags: [{title: 'ensaio', color: 'red'}]
			},
			{
				id: 3,
				subject: '[Manutenção] - OS93832',
				description: 'Observações',
				status: 'progress',
				owner: 2,
				type: '',
				tags: [{title: 'manutenção', color: 'white'}]
			},
			{
				id: 3,
				subject: '[Alinhamento] - OS999',
				description: 'Observações',
				status: 'done',
				owner: 2,
				type: '',
				tags: [{title: 'borracharia', color: 'green'}]
			}],
			members: [
				{
					id: 1,
					name: 'Beatriz'
				},
				{
					id: 2,
					name: 'Douglas'
				}
			],
			options: {
				defaultTaskDialog: true,
				defaultConfirmDialog: true,
				defaultMemberDialog: true
			}
		}
	},
	methods: {
		updateTask (task) {
			console.log(task)
		},
		deleteTask (task) {
			console.log(task)
		},
		deleteTag (task, tag) {
			console.log(task, tag)
		},
		addTag (task, tag) {
			console.log(task, tag)
		},
		updateMember (member) {
			this.members.push(Object.assign({}, member))
		}
	}
}
</script>
