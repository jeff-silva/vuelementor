<template>
    <div>
        <draggable v-model="props.value" class="list-group" handle=".vuelementor-handle"  @end="emitValue()">
            <div class="list-group-item d-flex align-items-center p-0" v-for="item in props.value">
                <div class="px-1"><div class="vuelementor-handle"></div></div>
                <div class="flex-grow-1"><slot name="item" :item="item">{{ item }}</slot></div>
                <div class="px-1" v-if="canDelete"><a href="javascript:;" @click="itemDelete(item)">&times;</a></div>
            </div>
        </draggable>
    </div>
</template>

<script>
import draggable from 'vuedraggable';

export default {
    props: {
        value: Array,
        canDelete: [Boolean, String],
    },

    components: { draggable },

    data() {
        return {
            props: JSON.parse(JSON.stringify(this.$props)),
        };
    },

    methods: {
        emitValue(value=null) {
            if (value!==value) this.props.value = value;
            this.$emit('value', this.props.value);
            this.$emit('input', this.props.value);
            this.$emit('change', this.props.value);
        },

        itemDelete(item) {
            if (typeof this.canDelete == 'string') {
                if (! confirm(this.canDelete)) return;
            }

            let index = this.props.value.indexOf(item);
            this.props.value.splice(index, 1);
            this.emitValue();
        },
    },
}
</script>