<template>
    <div class="vuelementor" :class="{'vuelementor-editable':editable}">
        <div class="vuelementor-content">
            <section v-for="section in props.value.sections" @click="sectionEdit(section)" :class="{'vuelementor-section-selected':section==sectionEditData}">
                <div v-bind="section.bind" class="row">
                    <div v-for="row in section.rows" class="col" @click="rowEdit(row)" :class="{'vuelementor-row-selected':row==rowEditData}">
                        <div v-for="comp in row.components" @click="componentEdit(comp)" :class="{'vuelementor-component-selected':comp==componentEditData}">
                            <pre>{{ comp }}</pre>
                            <component :is="comp.component" v-bind="comp.bind"></component>
                        </div>
                    </div>
                </div>
            </section>
        </div>

        <div class="vuelementor-editor" v-if="editable" style="max-width: 400px;">
            <el-tabs value="sections" type="border-card">
                <el-tab-pane name="sections" label="Seções" style="padding: 0 !important;">
                    <vuelementor-draggable v-model="props.value.sections" can-delete="Deseja deletar este item?" @change="emitValue()">
                        <template #item="{item}">
                            <input type="text" class="form-control border-0" v-model="item.label">
                        </template>
                    </vuelementor-draggable>
                </el-tab-pane>

                <el-tab-pane name="components" label="Components">
                    <div class="list-group">
                        <div class="list-group-item" v-for="comp in components">
                            {{ comp.label }}
                        </div>
                    </div>
                </el-tab-pane>

                <el-tab-pane name="component" label="Componente" v-if="componentEditData">
                    <input type="text" class="form-control" v-model="componentEditData.label">
                    <pre>{{ componentEditData }}</pre>
                </el-tab-pane>

                <el-tab-pane name="row" label="Row" v-if="rowEditData">
                    <input type="text" class="form-control" v-model="rowEditData.label">
                    <pre>{{ rowEditData }}</pre>
                </el-tab-pane>

                <el-tab-pane name="section" label="Seção" v-if="sectionEditData">
                    <el-checkbox v-model="sectionEditData.bind.class.container"> Container centralizado</el-checkbox>
                    
                    <div class="mt-2">Rows</div>
                    <vuelementor-draggable v-model="sectionEditData.rows" can-delete="Deseja deletar este item?" @change="emitValue()">
                        <template #item="{item}">
                            <input type="text" class="form-control border-0" v-model="item.label">
                        </template>
                    </vuelementor-draggable>

                    <pre>{{ sectionEditData }}</pre>
                </el-tab-pane>
            </el-tabs>
            <pre>{{ $data }}</pre>
        </div>
    </div>
</template>

<script>
import 'bootstrap/dist/css/bootstrap.min.css';

import Vue from 'vue';
import 'element-ui/lib/theme-chalk/index.css';
import ElementUI from 'element-ui';
Vue.use(ElementUI);

export default {
    props: {
        value: Object,
        editable: Boolean,
    },
    
    data() {
        let data = {};
        data.props = this.getProps(this.$props);
        data.sectionEditData = false;
        data.rowEditData = false;
        data.componentEditData = false;

        data.components = [];
        data.components.push((() => {
            let item = {};
            item.label = "Card";
            item.params = {
                header: "",
                body: "",
                footer: "",
            };
            item.component = {
                props: {header:String, body:String, footer:String},
                template: `<div class="card">
                    <div class="card-header">{{ header }}</div>
                    <div class="card-body">{{ body }}</div>
                    <div class="card-footer">{{ footer }}</div>
                </div>`,
            };
            item.componentEdit = {
                props: {header:String, body:String, footer:String},
                template: `<div>
                    <input type="text" class="form-control" v-model="header" />
                    <input type="text" class="form-control" v-model="body" />
                    <input type="text" class="form-control" v-model="footer" />
                </div>`,
            };
            return item;
        })());

        return data;
    },

    methods: {
        emitValue() {
            this.$emit('value', this.props.value);
            this.$emit('change', this.props.value);
        },

        uuid(prefix='') {
            return (prefix? `${prefix}-`: '')+ ([1e7] + 0).replace(/[018]/g, c => (c ^ (crypto.getRandomValues(new Uint8Array(1))[0] & (15 >> (c / 4)))).toString(16));
        },

        itemDefault(item={}, type='') {
            let uuid = this.uuid(type);
            return {
                id: uuid,
                label: uuid,
                bind: {
                    class: {},
                },
                ...(item||{})
            };
        },

        getProps(props={}) {
            props = JSON.parse(JSON.stringify(props));
            props.value = this.getValue(props.value || {});
            return props;
        },

        getValue(value={}) {
            value = {
                sections: [],
                ...value
            };

            value.sections = value.sections.map(section => {
                section = this.itemDefault(section, 'section');
                section.bind.class.container = section.bind.class.container || true;
                section.rows = section.rows || [];

                section.rows = section.rows.map(row => {
                    row = this.itemDefault(row, 'row');
                    row.components = row.components || [];

                    row.components = row.components.map(comp => {
                        comp = this.itemDefault(comp, 'component');
                        comp.component = comp.component || false;

                        return comp;
                    });

                    return row;
                });

                return section;
            });

            return value;
        },

        sectionEdit(section) {
            if (!this.editable) return;
            this.sectionEditData = section;
            this.rowEditData = false;
            this.componentEditData = false;
        },

        rowEdit(row) {
            if (!this.editable) return;
            setTimeout(() => {
                this.rowEditData = row;
                this.componentEditData = false;
            }, 10);
        },

        componentEdit(component) {
            if (!this.editable) return;
            setTimeout(() => {
                this.componentEditData = component;
            }, 20);
        },
    },
}
</script>

<style>
.vuelementor {
    display: flex;
    position: relative;
}

.vuelementor-handle {cursor: pointer;}
.vuelementor-handle:after {content: ":::";}

.vuelementor-content {flex-grow:1;}

/* Editables after padrão
.vuelementor-editable .vuelementor-section-selected:before,
.vuelementor-editable .vuelementor-row-selected:before,
.vuelementor-editable .vuelementor-component-selected:before {
    font-size: 10px;
    position: absolute;
    top: -20px;
    background: blue;
    color: #fff;
    padding:2px 5px;
    border-radius: 5px 5px 0px 0px;
}
*/

.vuelementor-editable .vuelementor-section-selected {outline: dashed 2px #666;}
/* .vuelementor-editable .vuelementor-section-selected:before {content:"Content";} */

.vuelementor-editable .vuelementor-row-selected {outline: dashed 2px #666;}
/* .vuelementor-editable .vuelementor-row-selected:before {content:"Row"} */

.vuelementor-editable .vuelementor-component-selected {outline: dashed 2px #666;}
/* .vuelementor-editable .vuelementor-component-selected:before {content:"Componente"} */

.vuelementor-editor {}
</style>