<template>
        <div class="search-div">
            <v-combobox
                ref="combobox"
                prepend-icon="mdi-magnify"
                v-model="selectedLibrary"
                :items="this.libraries"
                :item-text="library => `${library.name}`"
                label="Search"
                item-value="library"
                :search-input.sync="updateSearchInput"
                @change="searchLibrary"
                @update:search-input="onInputChange"
                @keydown.enter="handleKeydown"
                :auto-select-first="false"
                :clearable="true"
                color="#4E4A99"
                background-color="white"
            ></v-combobox>
        </div>
</template>
  
<script>
export default {
    name: 'SearchBar2',
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
        };
    },
    computed: {
        // This is the computed property that will filter the items based on the search query
        updateSearchInput: {
            // get() {
            //     // Read the value of the computed property
            //     if (this.searchInput != null) {
            //     return this.searchInput;
            //     }
            //     return "";
            // },
            set(value) {
                // Catch the case, if there is no library (null), so no error is displayed in the console
                console.log("Updating search input: ", value);
                return this.search = value;
            },
            get() {
                return this.search;
            },
        },
    },
    methods: {
        searchLibrary() {
            if (this.selectedLibrary == null || this.selectedLibrary === "") {
                this.$emit('search-change', '');
            }
            if (this.selectedLibrary != null) {
                console.log("Selected Library: ", this.selectedLibrary);
                this.$emit('search-library', this.selectedLibrary);
            }
        },
        onInputChange() {
            this.$emit('search-change', this.search);
        },
        handleKeydown() {
            if (this.libraries.includes(this.selectedLibrary && !this.search)) {
                this.$emit('search-change', this.search);
                this.$refs.combobox.blur();
            }
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