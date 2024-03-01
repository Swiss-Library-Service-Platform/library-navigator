<template>
    <v-toolbar floating rounded>
        <v-autocomplete v-model="selectedLibrary" :items="items" label="Search Libraries" item-text="displayText"
            item-value="library" :search-input.sync="search" @click="onLibrarySelected" :auto-select-first="true"
            :clearable="true" @click:clear="resetMap">
        </v-autocomplete>
        <v-btn @click="search" icon>
            <v-icon>mdi-magnify</v-icon>
        </v-btn>
        <v-btn icon v-show="showDeleteBtn">
            <v-icon @click="resetMap">mdi-crosshairs-gps</v-icon>
        </v-btn>
    </v-toolbar>
</template>
  
<script>
import jsonData from '@/assets/exampleLibraries.json';

export default {
    data() {
        return {
            items: jsonData.map(library => ({
                library,
                displayText: `${library.name}, ${library.city}` // Customize as needed
            })),
            selectedLibrary: null,
            search: '',
            showDeleteBtn: ((this.selectedLibrary != null) ? true : false),
        };
    },
    computed: {
        // This is the computed property that will filter the items based on the search query
        filteredItems() {
            if (!this.search) return this.items;
            const lowerCaseSearch = this.search.toLowerCase();
            return this.items.filter(item => {
                return (
                    item.library.name.toLowerCase().includes(lowerCaseSearch) ||
                    item.library.library_code.toLowerCase().includes(lowerCaseSearch) ||
                    item.library.street.toLowerCase().includes(lowerCaseSearch) ||
                    item.library.postal_code.toLowerCase().includes(lowerCaseSearch) ||
                    item.library.city.toLowerCase().includes(lowerCaseSearch)
                );
            });
        },

    },
    methods: {
        onLibrarySelected() {
            if (this.selectedLibrary != null) {
                console.log(this.selectedLibrary);
                // Access the entire JSON object of the selected library using this.selectedItem.library
                this.$emit('search-change', this.selectedLibrary);
            }
        },
        resetMap() {
            console.log("SEARCH: Reset Map");
            this.$emit('search-reset', null);
        },
    },
};
</script>
  
<style>
.v-toolbar {
    position: absolute;
    top: 25px;
    left: 50px;
    z-index: 1000;
}

.v-autocomplete {
    top: 10px;
    width: 300px;
}
</style>