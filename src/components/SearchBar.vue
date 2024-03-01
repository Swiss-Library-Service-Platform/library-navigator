<template>
    <v-toolbar floating rounded>
        <v-icon>mdi-magnify</v-icon>
        <v-autocomplete 
            v-model="selectedLibrary" 
            :items="filteredItems" 
            label="Search for Libraries"
            item-text="displayText"
            item-value="library" 
            :search-input.sync="updateSearchInput"
            @change="searchLibrary"
            :auto-select-first="true"
            :clearable="true"
            background-color="white"
            color="#4E4A99"
        ></v-autocomplete>
<!--         <v-btn icon>
            <v-icon @click="resetMap">mdi-home</v-icon>
        </v-btn> -->
    </v-toolbar>
</template>
  
<script>
import jsonData from '@/assets/data.json';

export default {
    data() {
        return {
            items: jsonData.map(library => ({
                library,
                displayText: `${library.name}, ${library.city}, ${library.library_code}`
            })),
            selectedLibrary: null,
            search: '',
        };
    },
    props: {
        searchInput: {
            type: String,
            default: '',
        },
        libraries: {
            type: Array,
            default: () => [],
        },
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
        updateSearchInput: {
            get() {
                // Read the value of the computed property
                if (this.searchInput != null) {
                return this.searchInput;
                }
                return "";
            },
            set(value) {
                // Catch the case, if there is no library (null), so no error is displayed in the console
                this.search = value;
            },
        }
    },
    methods: {
        searchLibrary() {
            if (this.selectedLibrary != null) {
                this.$emit('search-library', this.selectedLibrary);
            }
        },
        resetMap() {
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