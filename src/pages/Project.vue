<script setup>
import axios from 'axios'
import Cookies from 'js-cookie'
import { computed, onMounted, ref } from 'vue'
import RemoveProject from '@/components/Projects/Project/RemoveProject.vue'
import ResourceCardList from '@/components/Projects/Resource/ResourceCardList.vue'
import CreateResource from '@/components/Projects/Resource/CreateResource.vue'
import RemoveResource from '@/components/Projects/Resource/RemoveResource.vue'
import SettingsProject from '@/components/Projects/Project/SettingsProject.vue'
import SettingsResource from '@/components/Projects/Resource/SettingsResource.vue'

const items_url = `${import.meta.env.VITE_API_URL}/projects/`
const token = Cookies.get('token')

const props = defineProps({
	id: {
		type: Number,
		required: true,
	},
})

const project = ref({
	name: '',
	projectId: '',
})

const isRemoveProjectOpen = ref(false)
const isCreateResourceOpen = ref(false)
const isRemoveResourceOpen = ref(false)
const isSettingsProjectOpen = ref(false)
const isResourceProjectOpen = ref(false)
const projectIdToRemove = ref(null)
const projectId = ref(null)
const resourceIdToRemove = ref(null)
const isNotFound = ref(false)

const fetchProject = async () => {
	try {
		const response = await axios.get(`${items_url}${props.id}`, {
			headers: {
				Authorization: `Bearer ${token}`,
			},
		})
		project.value = response.data
	} catch (err) {
		console.log(err)
		isNotFound.value = true
	}
}

const openRemoveProject = id => {
	projectIdToRemove.value = id
	isRemoveProjectOpen.value = true
}

const closeRemoveProject = () => {
	isRemoveProjectOpen.value = false
}

const openCreateResource = () => {
	projectId.value = props.id
	isCreateResourceOpen.value = true
}

const closeCreateResource = () => {
	isCreateResourceOpen.value = false
}

const openRemoveResource = resourceId => {
	projectId.value = props.id
	resourceIdToRemove.value = resourceId
	isRemoveResourceOpen.value = true
}

const closeRemoveResource = () => {
	isRemoveResourceOpen.value = false
}

const openSettingsProject = () => {
	projectId.value = props.id
	isSettingsProjectOpen.value = true
}

const closeSettingsProject = () => {
	isSettingsProjectOpen.value = false
}

const openSettingsResource = resourceId => {
	projectId.value = props.id
	resourceIdToRemove.value = resourceId
	isResourceProjectOpen.value = true
}

const closeSettingsResource = () => {
	isResourceProjectOpen.value = false
}

const iconColor = computed(() => {
	const firstLetter = project.value.name
		? project.value.name.charAt(0).toUpperCase()
		: ''
	if ('ABC'.includes(firstLetter)) return '#ff8a00'
	if ('DEF'.includes(firstLetter)) return '#ff6b6b'
	if ('GHI'.includes(firstLetter)) return '#4CAF50'
	if ('JKL'.includes(firstLetter)) return '#2196F3'
	if ('MNO'.includes(firstLetter)) return '#9C27B0'
	if ('PQR'.includes(firstLetter)) return '#FF5722'
	if ('STU'.includes(firstLetter)) return '#795548'
	return '#607D8B'
})

const firstLetter = computed(() => {
	return project.value.name ? project.value.name.charAt(0).toUpperCase() : ''
})

const items = ref([])
const list = ref(true)

const fetchItems = async () => {
	try {
		const response = await axios.get(`${items_url}${props.id}/resource`, {
			headers: {
				Authorization: `Bearer ${token}`,
			},
		})
		items.value = response.data
		list.value = items.value.length > 0
	} catch (error) {
		console.error('Error fetching projects:', error)
	}
}

onMounted(async () => {
	await fetchProject()
	await fetchItems()
})
</script>

<template>
	<div class="main" v-if="!isNotFound">
		<SettingsResource
			v-if="isResourceProjectOpen"
			:id="projectId"
			:resourceId="resourceIdToRemove"
			@close="closeSettingsResource"
		/>
		<SettingsProject
			v-if="isSettingsProjectOpen"
			:projectId="projectId"
			@close="closeSettingsProject"
		/>
		<CreateResource
			v-if="isCreateResourceOpen"
			:id="projectId"
			@close="closeCreateResource"
		/>
		<RemoveProject
			v-if="isRemoveProjectOpen"
			:id="projectIdToRemove"
			@close="closeRemoveProject"
		/>
		<RemoveResource
			v-if="isRemoveResourceOpen"
			:id="projectId"
			:resourceId="resourceIdToRemove"
			@close="closeRemoveResource"
		/>
		<div class="project">
			<div class="container">
				<div class="menu">
					<router-link to="/projects" class="back-btn">
						<img src="/icons/arrow-icon.svg" alt="arrow" />
						Назад
					</router-link>
					<div class="panel">
						<button class="create-resource-btn" @click="openCreateResource">
							<img src="/icons/plus-icon.svg" alt="plus" />
							Создать ресурс
						</button>
						<button class="settings-btn" @click="openSettingsProject">
							<img src="/icons/settings-icon.svg" alt="settings" />
						</button>
						<button
							class="delete-btn"
							@click="openRemoveProject(project.projectId)"
						>
							<img src="/icons/delete-icon.svg" alt="delete" />
						</button>
					</div>
				</div>
				<div class="info">
					<div class="name-and-icon">
						<div class="icon" :style="{ backgroundColor: iconColor }">
							<span>{{ firstLetter }}</span>
						</div>
						<h1>{{ project.name }}</h1>
					</div>
					<div class="id">
						<p>ID: {{ project.projectId }}</p>
					</div>
				</div>
				<div class="resource-list" v-if="list">
					<ResourceCardList
						:items="items"
						@remove-resource="openRemoveResource"
						@settings-resource="openSettingsResource"
						:projectId="project.projectId"
					/>
				</div>
				<div class="NoneList" v-else>
					<div class="NoneListInfo">
						<img src="/public/icons/package-icon.svg" alt="package" />
						<h1>Список ресурсов пока пуст</h1>
						<p>У вас пока нет ресурсов. Создайте новый ресурс</p>
					</div>
				</div>
			</div>
		</div>
	</div>

	<div class="not-found" v-else>
		<div class="not-found-container">
			<h1>404</h1>
			<div class="not-found-text">
				<p>Извините, ваш проект не был найден.</p>
				<p>Пожалуйста проверьте правильность ссылки.</p>
			</div>
		</div>
	</div>
</template>

<style scoped>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Montserrat', sans-serif;
}
.container {
	max-width: 1240px;
	margin: 0 auto;
	padding: 20px;
	min-height: 60vh;
	display: flex;
	flex-direction: column;
}

.not-found-container {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	min-height: 65vh;
}

.not-found-text {
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	color: #e0e0e0;
	font-size: 18px;
	font-weight: 500;
	gap: 3px;
}

.not-found-container h1 {
	color: white;
	font-weight: 600;
	font-size: 74px;
}

.resource-list {
	width: 100%;
}

.menu {
	display: flex;
	justify-content: space-between;
	align-items: center;
	width: 100%;
	margin-bottom: 30px;
}

.back-btn {
	display: flex;
	align-items: center;
	gap: 10px;
	color: #ffffff;
	font-size: 18px;
	cursor: pointer;
	transition: color 0.3s ease;
	text-decoration: none;
}

.back-btn:hover {
	color: #ff8a00;
	filter: brightness(0) saturate(100%) invert(64%) sepia(50%) saturate(2161%)
		hue-rotate(343deg) brightness(97%) contrast(97%);
}

.panel {
	display: flex;
	align-items: center;
	gap: 20px;
}

.panel button {
	display: flex;
	align-items: center;
	gap: 10px;
	background-color: transparent;
	border: none;
	cursor: pointer;
	transition: transform 0.3s ease;
}
.panel img {
	width: 30px;
	height: 30px;
	user-select: none;
}

.panel button:hover {
	transform: translateY(-3px);
}

.settings-btn {
	border: 1px solid white !important;
	border-radius: 15px !important;
	padding: 10px 15px !important;
	box-shadow: 0 4px 20px white;
}

.delete-btn {
	border: 1px solid red !important;
	border-radius: 15px !important;
	padding: 10px 15px !important;
	box-shadow: 0 4px 20px red;
}

.create-resource-btn {
	background: linear-gradient(90deg, #ff8a00, #ff6b6b);
	color: white;
	padding: 12px 20px;
	border-radius: 15px;
	font-size: 16px;
	transition: background 0.4s ease, transform 0.3s ease, box-shadow 0.3s ease;
	box-shadow: 0 8px 20px rgba(255, 107, 107, 0.5);
}
.create-resource-btn:hover {
	background: linear-gradient(90deg, #ff7a00, #ff5656);
	transform: translateY(-4px);
	box-shadow: 0 10px 25px rgba(255, 107, 107, 0.7);
}

.name-and-icon {
	display: flex;
	align-items: center;
	color: white;
}

.icon {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 60px;
	height: 60px;
	border-radius: 50%;
	font-size: 24px;
	font-weight: bold;
	color: white;
	margin-right: 10px;
	user-select: none;
}

.info {
	display: flex;
	flex-direction: column;
	gap: 15px;
}
.id {
	color: #dddddd;
	margin-left: 5px;
}

.NoneList {
	display: flex;
	align-items: center;
	justify-content: center;
	max-width: 1240px;
	margin: 0 auto;
	min-height: 50vh;
}

.NoneListInfo {
	display: flex;
	align-items: center;
	justify-content: center;
	flex-direction: column;
	gap: 20px;
	color: white;
}

@media (max-width: 768px) {
	.not-found-container {
		width: 100%;
	}
	.not-found-text {
		font-size: 16px;
	}
	.container {
		align-items: center;
	}
	.create-resource-btn {
		width: 100%;
		padding: 10px;
		font-size: 14px;
	}
	.panel img {
		width: 25px;
		height: 25px;
	}
	.menu {
		flex-direction: column;
		gap: 10px;
	}

	.project-card h1 {
		font-size: 26px;
	}
}
</style>
