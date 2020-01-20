<template>
    <div> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_page_header" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container position_relative">
                        <h1>Location</h1>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-12">
                            <breadcrumb></breadcrumb>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-12">
                            <div v-if="main" v-html="main.body"></div>
                        </div>
                    </div>
                </div>
                <div class="main_container">
                    <div class="location_map">
                        <p class="visuallyhidden">
                            <a :href="'https://maps.google.com/?q=' + property.name">View {{ property.name }} on Google Maps</a>.
                        </p>
                        <iframe aria-hidden="true" :title="'Google Map for ' + property.name" :src="propertyAddress()" width="100%" height="490" frameborder="0" style="border:0" allowfullscreen></iframe>
                    </div>
                </div>
                <div class="main_container">
                    <div class="row">
                        <div class="col-md-6">
                            <div v-if="address" v-html="address.body"></div>
                        </div>
                        <div class="col-md-6">
                            <div v-if="directions" v-html="directions.body"></div>
                        </div>
                    </div>
                </div>
                <div class="location_image_container">
                    <div class="location_image" v-if="pageImages" v-for="item in pageImages">
                        <img :src="item.image_url" alt="item.id" class="img_max" />   
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
	define(["Vue", "vuex", "vue!google_map", "json!site.json"], function(Vue, Vuex, google_map, siteInfo ) {
		return Vue.component("location-component", {
            template: template, // the variable template will be injected
            data: function () {
                return {
                    dataLoaded: false,
                    pageBanner: null,
                    main: null,
                    address: null,
                    directions: null,
                    pageImages: null
                }
            },
            created() {
                this.loadData().then(response => {
                    var repo = this.findRepoByName('Location Banner').images;
                    if(repo != null) {
                        this.pageBanner = repo[0];
                    } else {
                        this.pageBanner = {
                            "image_url": "//codecloud.cdn.speedyrails.net/sites/5de7dca36e6f6435b2020000/image/jpeg/1529532304000/insidebanner2.jpg"
                        }
                    }
                    
                    var temp_repo = this.findRepoByName('Location Images');
                    if(temp_repo) {
                        this.pageImages = temp_repo.images;
                    }
                    this.main = response[1].data;
                    this.address = response[1].data.subpages[0]
                    this.directions = response[1].data.subpages[1]
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'repos',
                    'findRepoByName'
                ]),
                getPropertyAddress() {
                    return this.property.name + ' ' + this.property.address1 + ' ' + this.property.city + ' ' + this.property.country + ' ' +this.property.province_state + ' ' + this.property.province_state
                }
            },
            methods: {
                loadData: async function () {
                    this.property.mm_host = this.property.mm_host.replace("http:", "");
                    try {
                        let results = await Promise.all([this.$store.dispatch("getData", "repos"), this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/" + siteInfo.subdomain + "-location.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                propertyAddress() {
                    var address = this.property.name + "+" + this.property.address1 + "+" + this.property.city + "+" + this.property.province_state + "+" + this.property.country + this.property.postal_code
                    var key ="AIzaSyCukCjH3fsuDYBdI44hZKL43m60jEToJjY"
                    var src = "https://www.google.com/maps/embed/v1/place?q=" + address + "&key="+ key
                    return src
                }
            }
        });
	});
</script>