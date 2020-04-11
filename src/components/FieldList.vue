<template>
    <table class="table">
        <caption>
            <h3>Field</h3>
        </caption>
        <thead>
            <tr>
                <th width="130px"></th>
                <th>Name</th>
                <th>Type</th>
                <th>Length / Enum</th>
                <th>Default / Comment</th>
            </tr>
        </thead>
        <tbody>
            <tr v-for="field in manager.list" :key="field.name">
                <td class="button-cell">
                    <b-button-group>
                        <b-button @click="remove(field)" variant="outline-danger"> - </b-button>
                        <b-button @click="manager.moveUp(field)" variant="outline-primary"> ↑ </b-button>
                        <b-button @click="manager.moveDown(field)" variant="outline-primary"> ↓ </b-button>
                    </b-button-group>
                </td>
                <td>
                    <b-button @click="rename(field)" variant="outline-primary"> {{ field.name }} </b-button>
                </td>
                <td>
                    <b-button @click="setType(field)" variant="outline-primary"> {{ field.type }} </b-button>
                </td>
                <td>
                    <input v-if="field.hasLength" v-model="field.length" class="form-control" type="text" />
                </td>
                <td>
                    <FieldPanel :field="field"></FieldPanel>
                </td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td>
                    <select v-model="selected" @change="addType($event.target.value)" class="form-control">
                        <option selected="true" disabled="disabled" value=""> ---- </option>
                        <option v-for="name in CommonTypeList" :key="name">{{ name }}</option>
                    </select>
                </td>
                <td>
                    <select @change="addField($event.target.value, IntegerFieldList)" class="form-control">
                        <option selected="true" disabled="disabled" value=""> ---- </option>
                        <option v-for="field in IntegerFieldList" :value="field.name" :key="field.name">
                            {{ field.name }}
                        </option>
                    </select>
                </td>
                <td>
                    <select @change="addField($event.target.value, CommonFieldList)" class="form-control">
                        <option selected="true" disabled="disabled" value=""> ---- </option>
                        <option v-for="field in CommonFieldList" :value="field.name" :key="field.name">
                            {{ field.name }}
                        </option>
                    </select>
                </td>
                <td>
                    <select @change="addName($event.target.value)" class="form-control">
                        <option selected="true" disabled="disabled" value=""> ---- </option>
                        <option v-for="entity in EntityList" :key="entity.name">{{ entity.tableName }}_id</option>
                    </select>
                </td>
                <td>
                    <b-button @click="add" variant="outline-primary"> + </b-button>
                </td>
            </tr>
        </tfoot>
    </table>
</template>

<script>
import FieldPanel from './FieldPanel.vue'
import { CommonFieldList, CommonTypeList, FieldTypeList, IntegerFieldList } from '../presets/field.js'
import builder from '../states/builder.js'
import dialogue from '../states/listdialogue.js'

export default {
    name: 'FieldList',
    components: { FieldPanel },
    props: {
        manager: {
            type: Object,
            required: true,
        },
    },
    data() {
        return {
            EntityList: builder.project.EntityManager.list,
            CommonFieldList,
            CommonTypeList,
            IntegerFieldList,
            selected: '',
        }
    },
    methods: {
        add() {
            dialogue.show(FieldTypeList, 'name', 'Select a Type', ok => {
                try {
                    const fff = this.manager.cloneType(dialogue.selected.type)
                    this.manager.add(fff)
                } catch (error) {
                    console.error(error)
                    alert(error)
                }
            })
        },
        addField(name, list) {
            try {
                const found = list.find(item => item.name === name)
                const fff = this.manager.make(found.name, found.type)
                fff.load(found)
                this.manager.add(fff)
            } catch (error) {
                console.error(error)
                alert(error)
            }
        },
        addName(name) {
            try {
                const fff = this.manager.make(name, 'integer')
                this.manager.add(fff)
            } catch (error) {
                console.error(error)
                alert(error)
            }
        },
        addType(type) {
            try {
                this.selected = ''
                const field = this.manager.cloneType(type)
                this.manager.add(field)
                this.rename(field)
            } catch (error) {
                console.error(error)
                alert(error)
            }
        },
        remove(field) {
            if (confirm('Are you sure?')) {
                this.manager.remove(field)
            }
        },
        rename(field) {
            const name = prompt('Please input the name', field.name)
            if (name) {
                try {
                    field.name = name
                } catch (error) {
                    console.error(error)
                    alert(error)
                }
            }
        },
        setType(field) {
            dialogue.show(FieldTypeList, 'name', 'Select a Type', ok => {
                field.type = dialogue.selected.type
            })
        },
    },
}
</script>