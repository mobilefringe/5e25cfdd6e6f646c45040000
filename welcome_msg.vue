<template>
    <div v-if="property" class="main_container">
        <div class="message_content">
            <h1 class="center" v-if="property.welcome_message">{{ property.welcome_message }}</h1>
            <p class="center" v-if="property.description">{{ property.description }}</p> 
        </div>
    </div>
</template>

<script>
    define(["Vue", "vuex"], function(Vue, Vuex){
        return Vue.component("messages-component", {
            template: template, // the variable template will be injected
            computed: {
                ...Vuex.mapGetters([
                    'property'
                ])
            },
            created(){
                console.log(this.property)
            }
        })
    });
</script>