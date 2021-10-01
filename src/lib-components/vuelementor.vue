<template>
    <div class="vuelementor" :class="{'vuelementor-editable':editable}">
        <div class="vuelementor-content">
           
            <section class="vuelementor-empty" v-if="editable && !props.value.sections.length">
                Adicionar seção
            </section>

            <section v-for="section in props.value.sections" @click="sectionEdit(section)" :class="{'vuelementor-section-selected':(sectionEditData && sectionEditData.id==section.id)}">
                <div v-bind="section.bind" class="row g-0">
                    <div class="vuelementor-empty" v-if="editable && !section.columns.length">
                        Adicionar coluna
                    </div>

                    <div v-for="column in section.columns" v-bind="column.bind" class="col" @click="columnEdit(column)" :class="{'vuelementor-column-selected':(columnEditData && columnEditData.id==column.id)}">

                        <div class="vuelementor-empty" v-if="editable && !column.components.length">
                            Adicionar componente
                        </div>

                        <div v-for="comp in column.components" @click="componentEdit(comp)" :class="{'vuelementor-component-selected':(componentEditData && componentEditData.id==comp.id)}">
                            <pre v-bind="comp.bind">{{ comp }}</pre>
                            <!-- <component :is="comp.component" v-bind="comp.bind"></component> -->
                        </div>
                    </div>
                </div>
            </section>
        </div>

        <div class="vuelementor-editor bg-light shadow-sm" v-if="editable" style="max-width: 400px;">
            <el-collapse value="" accordion>
                <el-collapse-item name="component" :title="`Componente: ${componentEditData.label}`" v-if="componentEditData">
                    <vuelementor-style v-model="componentEditData" @change="emitValue()"></vuelementor-style>
                </el-collapse-item>

                <el-collapse-item name="column" :title="`Coluna: ${columnEditData.label}`" v-if="columnEditData">
                    <vuelementor-style v-model="columnEditData" @change="emitValue()"></vuelementor-style>
                </el-collapse-item>

                <el-collapse-item name="section" :title="`Seção: ${sectionEditData.label}`" v-if="sectionEditData">
                    <!-- <el-checkbox v-model="sectionEditData.bind.class.container"> Container centralizado</el-checkbox> -->

                    <vuelementor-style v-model="sectionEditData" @change="emitValue()"></vuelementor-style>
                    
                    <div class="mt-2">Colunas</div>
                    <vuelementor-draggable v-model="sectionEditData.columns" can-delete="Deseja deletar este item?" @change="emitValue()">
                        <template #item="{item}">
                            <input type="text" class="form-control border-0" v-model="item.label">
                        </template>
                    </vuelementor-draggable>
                </el-collapse-item>

                <el-collapse-item title="Componentes" name="components">
                    <div class="list-group">
                        <div class="list-group-item" v-for="comp in components">
                            {{ comp.label }}
                        </div>
                    </div>
                </el-collapse-item>

                <el-collapse-item title="Seções" name="secoes">
                    <vuelementor-draggable v-model="props.value.sections" can-delete="Deseja deletar este item?" @change="emitValue()">
                        <template #item="{item}">
                            <input type="text" class="form-control border-0" v-model="item.label">
                        </template>
                    </vuelementor-draggable>
                </el-collapse-item>
            </el-collapse>
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
        data.columnEditData = false;
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
        emitValue(value=null) {
            if (value!==value) this.props.value = value;
            this.$emit('value', this.props.value);
            this.$emit('input', this.props.value);
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
                    id: uuid,
                    class: "",
                    style: "",
                },
                style: "",
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
                // section.bind.class.container = section.bind.class.container || true;
                section.columns = section.columns || [];

                section.columns = section.columns.map(column => {
                    column = this.itemDefault(column, 'column');
                    column.components = column.components || [];

                    column.components = column.components.map(comp => {
                        comp = this.itemDefault(comp, 'component');
                        comp.component = comp.component || false;

                        return comp;
                    });

                    return column;
                });

                return section;
            });

            return value;
        },

        sectionEdit(section) {
            if (!this.editable) return;
            this.sectionEditData = section;
            this.columnEditData = false;
            this.componentEditData = false;
        },

        columnEdit(column) {
            if (!this.editable) return;
            setTimeout(() => {
                this.columnEditData = column;
                this.componentEditData = false;
            }, 5);
        },

        componentEdit(component) {
            if (!this.editable) return;
            setTimeout(() => {
                this.componentEditData = component;
            }, 10);
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

.vuelementor-empty {
    cursor: pointer;
    border: dashed 3px #ccc;
    padding: 30px 0px;
    margin: 5px 0;
    color: #ccc;
    text-align: center;
}

.vuelementor-content {flex-grow:1;}
.vuelementor-editable .vuelementor-content  {padding-right: 15px;}

/* Editables after padrão
.vuelementor-editable .vuelementor-section-selected:before,
.vuelementor-editable .vuelementor-column-selected:before,
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

.vuelementor-editable .vuelementor-section-selected {box-shadow: 0 0 0 1px var(--bs-primary);}
/* .vuelementor-editable .vuelementor-section-selected:before {content:"Content";} */

.vuelementor-editable .vuelementor-column-selected {box-shadow: 0 0 0 2px var(--bs-primary);}
/* .vuelementor-editable .vuelementor-column-selected:before {content:"Row"} */

.vuelementor-editable .vuelementor-component-selected {outline: dashed 2px #666;}
/* .vuelementor-editable .vuelementor-component-selected:before {content:"Componente"} */

.vuelementor-editor {}

/* ui-elements */
.vuelementor .el-collapse-item {background:transparent;}
.vuelementor .el-collapse-item__header {background:transparent; padding-left:15px;}
.vuelementor .el-collapse-item__header.is-active {background:var(--bs-dark); color:#fff;}
.vuelementor .el-collapse-item__content {padding: 10px;}
</style>