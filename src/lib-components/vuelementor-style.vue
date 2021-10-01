<template>
    <div>
        <div class="input-group mb-2">
            <div class="input-group-text" style="width:80px;">Nome</div>
            <input type="text" class="form-control" v-model="value.label" @keyup="emitValue()">
        </div>
        
        <div class="input-group mb-2">
            <div class="input-group-text" style="width:80px;">ID</div>
            <input type="text" class="form-control" v-model="value.bind.id" @keyup="emitValue()">
        </div>

        <div class="input-group mb-2">
            <div class="input-group-text" style="width:80px;">Classes</div>
            <input type="text" class="form-control" v-model="value.bind.class" @keyup="emitValue()">
        </div>

        <div class="input-group mb-2">
            <div class="input-group-text" style="width:80px;">Style</div>
            <input type="text" class="form-control" v-model="value.bind.style" @keyup="emitValue()">
        </div>
    </div>
</template>

<script>
export default {
    props: {
        value: Object,
    },

    data() {
        return {
            // props: JSON.parse(JSON.stringify(this.$props)),
        };
    },

    methods: {
        emitValue(value=null) {
            // if (value!==value) this.props.value = value;
            // this.$emit('value', this.props.value);
            // this.$emit('input', this.props.value);
            // this.$emit('change', this.props.value);
        },
    },
}
</script>