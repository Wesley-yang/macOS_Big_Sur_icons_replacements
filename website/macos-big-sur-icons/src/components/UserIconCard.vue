<template>
    <div :label="icon.appName.replaceAll('_', ' ') + 'Icon'" class="card-wrapper card-hover coral-card m-0">
        <div class="m-auto">

            <!-- Icon image -->
            <div class="card-img-wrapper" style="max-width: 120px;">
                
                <!-- macOS icon download -->
                <a v-if="isMacOs" @click="addClickCount(icon)" rel="noopener" :href="icon.icnsUrl">
                    <div v-lazy-container="{ selector: 'img', loading: coralIcons.placeholderIcon }">
                    <img width="100px" height="100px" :alt="icon.appName +' icon'" :data-src="icon.lowResPngUrl">
                    </div>
                </a>

                <!-- iOS icon download -->
                <a v-else @click="addClickCount(icon)" target="_blank" rel="noopener" :href="icon.iOSUrl">
                    <div v-lazy-container="{ selector: 'img', loading: coralIcons.placeholderIcon }">
                        <img width="100px" height="100px" :alt="icon.appName +' icon'" :data-src="icon.lowResPngUrl">
                    </div>
                </a>
            </div>

            <!-- Icon meta -->
            <div label="Icon info" class="card-text-wrapper p-l-15 p-r-15">

                <!-- App name -->
                <h3 class="coral-font-color m-0">
                    <span>
                        {{icon.appName.replaceAll("_", " ")}}
                    </span>
                </h3>
                
                <!-- Credit -->
                <p class="coral-Body--XS opacity-70 m-b-5 p-t-5">
                    <router-link :to="'/u/'+icon.usersName" class="coral-Link">{{icon.usersName}}</router-link>
                    on
                    <span class="coral-Body--XS">
                        {{ getDate(icon.timeStamp) }}
                    </span>
                </p>
                
                <div v-if="isOwner" class="p-t-10 p-b-5">
                    <button @click="showDialog('editIconDialog')" is="coral-button" variant="outline">Edit</button>
                </div>

            </div>

        </div>
        <!-- Download Counter -->
        <div label="Download Counter" class="download-counter-wrapper opacity-70">
            <p v-if="icon.downloads > 1" class="coral-Body--XS m-0 d-inline">
                {{icon.downloads}}
            </p>

            <p v-else class="coral-Body--XS m-0 d-inline">
                0
            </p>

            <img height="10px" :src="coralIcons.download" alt="Download counter">
        </div>
    </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
import Parse from 'parse'

import deleteDialog from './deleteDialog.vue';
import EditIconDialog from "./EditIconDialog.vue"

var Icons = Parse.Object.extend("Icons2");

export default {
    name: "IconCard",
    
    props:{
        icon:{},
        isAdmin: false,
        isMacOs: true,
        isOwner: false
    },

    components: {
        EditIconDialog,
        deleteDialog
    },

    data: function(){
        return{
            coralIcons:{
                addIcon: require("../assets/icons/add.svg"),
                delete: require("../assets/icons/delete.svg"),
                newItem: require("../assets/icons/newItem.svg"),
                edit: require("../assets/icons/edit.svg"),
                download: require("../assets/icons/Download.svg"),
                placeholderIcon: require("../assets/placeholder-icon.png"),
            },
            dialog:{
                editIcon: false,
                deleteIcon: false
            }
        }
    },
    
    methods:{
        ...mapActions(['showEl', 'setSelectedIcon', 'addClickCount']),

        prettifyName(name){
            console.log("name: ", name);
            name = name.replaceAll("_", " ")
            return name
        },
        
        showDialog(id){
            let parent = this
            parent.setSelectedIcon(parent.icon)
            parent.showEl(id)
        },
        
        getDate(timeStamp){
            let newDate = new Date(timeStamp)
            
            let day = newDate.getUTCDate()
                if (day < 10) {
                day = "0"+day
                }
            let month = newDate.getUTCMonth() + 1
                if (month < 10) {
                month = "0"+month
                }
            let year = newDate.getFullYear()
            let date = day + "/" + month + "/" + year

            return date
        },

        async editDoc(icon, e, field){
            let newName = e.target.value
            
            let id
            if (icon.algoliaID == undefined) {
                id = icon.id
            } else {
                id = icon.algoliaID
            }

            console.log(icon);
            console.log(id);

            const IconsBase = Parse.Object.extend("Icons2");
            const query = new Parse.Query(IconsBase);
            const docToEdit = await query.get(id)


            docToEdit.set({ [field]: newName }) // Save icnsToStore obj with .icns file and its url to Parse server
            docToEdit.save().then(() =>{
                console.log(field, "updated.");
            }).catch((e) =>{
                document.getElementById("error").show()
            })
        },

    },

    computed:{
    }
}
</script>

<style>

</style>