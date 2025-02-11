<template>
    <div class="card-wrapper card-hover coral-card">

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
                <div class="placeholder-icon" v-lazy-container="{ selector: 'img', loading: coralIcons.placeholderIcon }">
                    <img width="100px" height="100px" :alt="icon.appName +' icon'" :data-src="icon.lowResPngUrl">
                </div>
            </a>
        </div>

        <!-- Icon meta -->
        <div class="card-text-wrapper p-l-16 p-r-16 p-b-16">

            <!-- App name -->
            <h3 class="coral-font-color m-b-0">
                <input v-if="isAdmin" class="editable-input m-b-0" @change="editDoc(icon, $event, 'appName')" type="text" variant="quiet" :value="prettifyName(icon.appName)" is="coral-textfield" aria-label="text input">
                
                <span>
                    {{icon.appName.replaceAll("_", " ")}}
                </span>
            </h3>

            <!-- User's name Admin-->
            <p v-if="isAdmin" class="coral-Body--XS p-b-0 opacity-80 m-b-0">
                <input class="editable-input" @change="editDoc(icon, $event, 'usersName')" type="text" variant="quiet" :value="icon.usersName" is="coral-textfield" aria-label="text input">
            </p>
            
            <!-- Credit Admin-->
            <p v-if="isAdmin" class="coral-Body--XS p-b-0 opacity-50 m-b-0">
                <input class="editable-input small-text" @change="editDoc(icon, $event, 'credit')" type="text" variant="quiet" :value="icon.credit" is="coral-textfield" aria-label="text input">
            </p>
            
            <!-- Credit -->
            <p v-else class="coral-Body--XS opacity-60 m-b-8">
                <router-link :to="'/u/'+icon.usersName" class="coral-Link">{{icon.usersName}}</router-link>
                on
                <span class="coral-Body--XS opacity-80">
                    {{ getDate(icon.timeStamp) }}
                </span>
            </p>
            
            <div v-if="isOwner" class="p-t-8 p-b-4">
                <button @click="showDialog('editIconDialog')" is="coral-button" variant="outline">Edit</button>
            </div>

        </div>


    </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';
import Parse from 'parse/dist/parse.min.js';

import deleteDialog from './deleteDialog.vue';
import EditIconDialog from "./EditIconDialog.vue"

var Icons = Parse.Object.extend("Icons2");

import addCoralIcon from "../assets/icons/add.svg"
import newItemCoralIcon from "../assets/icons/newItem.svg"
import editCoralIcon from "../assets/icons/edit.svg"
import placeholderCoralIcon from "../assets/placeholder-icon.png"
import deleteIcon from "../assets/icons/delete.svg"

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
                delete: deleteIcon,
                addIcon: addCoralIcon,
                newItem: newItemCoralIcon,
                edit: editCoralIcon,
                loading: placeholderCoralIcon,
            },
            dialog:{
                editIcon: false,
                deleteIcon: false
            }
        }
    },
    
    methods:{
        ...mapActions(['showEl', 'setSelectedIcon']),

        prettifyName(name){
            name = name.replaceAll("_", " ")
            return name
        },
        
        showDialog(id){
            let parent = this
            parent.setSelectedIcon(parent.icon)
            parent.showEl(id)
        },

        async addClickCount(icon){
            var id
            if (icon.id) {
                id = icon.id
            } else{
                id = icon.objectID
            }
            icon = { appName: icon.appName, id: id }
            await Parse.Cloud.run("addClickCount", {icon: icon})
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