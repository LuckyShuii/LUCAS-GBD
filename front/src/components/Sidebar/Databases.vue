<script setup>
import { ref, onMounted, watch } from 'vue'
import PulseLoader from 'vue-spinner/src/PulseLoader.vue'
import axios from 'axios'
import Swal from 'sweetalert2'

const props = defineProps(['openDb', 'newDbCreated'])
const emit = defineEmits(['getDbName'])

const loading = ref(true)

const activeDbIndex = ref(-1)
const activeTableIndex = ref(-1)

const databaseList = ref({})

const notify = (title, text, icon, timer) => {
    Swal.fire({
        title: title,
        text: text,
        icon: icon,
        timer: timer,
        position: 'top-right',
        toast: true,
        showConfirmButton: false,
        showCloseButton: true
    })
}

const getDatabaseList = async () => {
    try {
        databaseList.value = await axios.get("http://127.0.0.1:8000/api/databases");
    } catch (err) {
        notify('Erreur !', 'Une erreur est survenue lors de la récupération des bases de données.', 'error', 10000);
    }
}

const toggleTables = (index) => {
    activeDbIndex.value = activeDbIndex.value === index ? -1 : index;
}

const toggleColumns = (index) => {
    activeTableIndex.value = activeTableIndex.value === index ? -1 : index;
}

const setDbName = () => {
    if (activeDbIndex.value === -1) {
        emit('getDbName', '')
        return
    }
    emit('getDbName', databaseList.value.data.db_list[activeDbIndex.value].Database)
}

onMounted(async () => {
    await getDatabaseList()
    loading.value = false
})

watch(() => props.newDbCreated, async () => {
    console.log('newDbCreated', props.newDbCreated)
    loading.value = true
    await getDatabaseList()
    loading.value = false
    activeDbIndex.value = -1
    activeTableIndex.value = -1
})

watch(() => props.openDb, async () => {
    activeDbIndex.value = -1
    activeTableIndex.value = -1
})
</script>

<template>
    <section id="loader" v-if="loading">
        <p>Chargement ...</p>
        <pulse-loader :loading="loading" :color="color" :size="size"></pulse-loader>
    </section>
    <section id="databases" v-if="!loading">
        <ul>
            <li v-for="(db, index) in  databaseList.data.db_list " :key="db.Database" class="db-item"
                @click="setDbName">
                <button class="db button-no-style" @click="toggleTables(index)">
                    <img src="../../assets/icons/menu-arrow.svg"
                        :class="activeDbIndex === index ? 'selected-arrow' : 'menu-arrow'" alt="arrow" />
                    <img src="../../assets/icons/db.svg" alt="Database" />
                    {{ db.Database }}
                </button>
                <button class="btn-add-table button-no-style">
                    <img src="../../assets/icons/add.svg" alt="Add" />
                    ajouter une table
                </button>
                <ul>
                    <li v-for="(table, index) in  db.tables " :key="table" v-if="activeDbIndex === index">
                        <button class="table button-no-style" @click="toggleColumns(index)"><img
                                src="../../assets/icons/list-arrow.svg" alt="Table" />
                            {{ table.table }}
                        </button>
                        <ul v-if="activeTableIndex === index">
                            <li v-for=" column  in  table.columns " :key="column">
                                <button class="column button-no-style"><img src="../../assets/icons/list-arrow.svg"
                                        alt="Column" />
                                    {{ column.Field }}
                                </button>
                            </li>
                        </ul>
                    </li>
                </ul>
            </li>
        </ul>
    </section>
</template>

<style scoped>
.db {
    margin: 0.5rem 0;
    font-weight: 400;
    width: 100%;
    display: flex;
    justify-content: flex-start;
}

#loader {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 10rem;
    flex-direction: column;
    text-align: center;
}

#loader p {
    margin-bottom: 1rem;
}

.db-item {
    margin: 1rem 0;
}

.menu-arrow {
    width: 5px;
}

.selected-arrow {
    width: 5px;
    margin-right: 10px;
    transform: rotate(90deg);
    transition: 0.3s;
}

.btn-add-table {
    font-size: 10px;
    color: black;
    margin-top: -10px;
}

.btn-add-table:hover {
    font-size: 11px;
    transition: 0.3s;
}

.table {
    margin-left: 1rem;
}

.column {
    margin-left: 2rem;
}

.table:hover,
.column:hover,
.db:hover {
    text-decoration: underline;
}
</style>
