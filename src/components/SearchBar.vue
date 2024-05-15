<template>
        <div class="search-div">
                <v-autocomplete
                    prepend-icon="mdi-magnify"
                    v-model="selectedLibrary" 
                    :items="filteredItems"
                    label="Search"
                    item-text="displayText"
                    item-value="library"
                    :search-input.sync="updateSearchInput"
                    @change="searchLibrary"
                    :auto-select-first="true"
                    :clearable="true"

                    color="#4E4A99"
                    background-color="white"
                ></v-autocomplete>
        </div>
</template>
  
<script>
export default {
    name: 'SearchBar',
    props: {
        libraries: {
            type: Array,
            default: () => [],
        },
        searchInput: {
            type: String,
            default: '',
        },
    },
    data() {
        return {
            items: [],
            selectedLibrary: null,
            search: '',
        };
    },
    computed: {
        filteredItems() {
            if (!this.libraries || this.libraries.length === 0) {
                return [];
            }
            var items = this.libraries.map(library => ({
                library,
                displayText: `${library.name} [${library.library_code}], ${library.city}`
            }));
            return items;
        },
        // This is the computed property that will filter the items based on the search query
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
        },
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
.search-div {
    border-radius: 10px;
    background-color: #FFFFFF;
    padding-right: 10px;
    padding-left: 10px;
}

.v-toolbar {
    position: absolute;
    top: 0px;
    left: 0px;
    z-index: 1000;
}

.v-autocomplete {
    top: 10px;
    width: 300px;
}
</style>