<script setup>
import Buttons from '../components/Main/Buttons.vue'
import Query from '../components/Main/Query.vue'
import Importer from '../components/Main/Importer.vue'
import Parametres from '../components/Main/Parametres.vue'
import Utilisateurs from '../components/Main/Utilisateurs.vue'
import Structure from '../components/Main/Structure.vue'
import CreateDB from '../components/Main/CreateDB.vue'
import { ref } from 'vue'

const props = defineProps(['dbName', 'createDB'])
const emit = defineEmits(['setCreateDB', 'setNewDbCreated'])

const activeTab = ref('structure')
const view = ref('database')

const changeTab = (event) => {
    activeTab.value = event
    emit('setCreateDB')
}

</script>

<template>
    <main>
        <CreateDB v-if="createDB" @setNewDbCreated="emit('setNewDbCreated')" />
        <div v-if="!createDB">
            <Buttons @active-tab="changeTab" :activeTab="activeTab" />
            <section class="db-check" v-if="!dbName">
                <h2 id="main-title">
                    <img src="../assets/icons/db.svg" alt="Add" />
                    Aucune base de données sélectionnée
                </h2>
            </section>
            <Structure v-if="activeTab === 'structure' && dbName" :view="view" :dbName="dbName"
                @updateActiveTab="activeTab = 'structure'" />
            <Query v-if="activeTab === 'query' && dbName" :dbName="dbName" @updateActiveTab="activeTab = 'structure'" />
            <Importer v-if="activeTab === 'importer' && dbName" />
            <Parametres v-if="activeTab === 'parametres' && dbName" :dbName="dbName"
                @updateActiveTab="activeTab = 'structure'" @setNewDbCreated="emit('setNewDbCreated', 'drop')" />
            <Utilisateurs v-if="activeTab === 'utilisateurs' && dbName" />
        </div>
    </main>
</template>

<style scoped>
main {
    margin-top: 1rem;
    margin-right: 1rem;
    width: calc(100% - 350px);
    height: calc(100vh - 100px);
}

.active {
    background-color: red;
}

.db-check {
    width: 100%;
    margin-top: 1rem;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 5px;
    resize: none;
    padding-bottom: 1rem
}

#main-title {
    display: flex;
    align-items: center;
    margin-top: 8px;
}
</style>