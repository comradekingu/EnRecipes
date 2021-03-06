<template>
<Page @loaded="onPageLoad" actionBarHidden="true" :androidStatusBarBackground="appTheme == 'Light' ? '#f1f3f5' : '#212529'">
  <RadSideDrawer allowEdgeSwipe="true" showOverNavigation="true" ref="drawer" id="sideDrawer" drawerContentSize="280" gesturesEnabled="true" drawerTransition="SlideInOnTopTransition">
    <GridLayout rows="*, auto" columns="*" ~drawerContent class="sd">
      <StackLayout row="0">
        <GridLayout rows="48" columns="auto, *, auto" v-for="(item, index) in topmenu" :key="index" class="sd-item orkm" :class="{
              'selected-sd-item': currentComponent === item.component,
            }">
          <MDRipple row="0" colSpan="3" @tap="navigateTo(item.component, item.component, false)" />
          <Label col="0" row="0" class="bx" :text="icon[item.icon]" />
          <Label col="1" row="0" :text="`${item.title}` | L" />
          <Label class="recipeCount" v-if="getRecipeCount(item.title)" :text="getRecipeCount(item.title)" col="2" />
        </GridLayout>
        <GridLayout class="sd-group-header orkm" rows="auto" columns="*, auto" v-if="cuisinesWithRecipes.length">
          <Label class="filterPath" verticalAlignment="center" col="0" :text="getCurrentPath | L" textWrap='true' />
          <MDButton :visibility="selectedCuisine?'visible':'hidden'" variant="text" @tap="previousRecipeFilter" class="bx" col="2" :text="icon.back" />
        </GridLayout>
        <ScrollView height="100%">
          <StackLayout>
            <GridLayout v-for="(item, index) in getRecipeList" :key="index" class="sd-item orkm" :class="{
                  'selected-sd-item': selectedTag == item,
                }" columns="auto, *, auto">
              <MDRipple row="0" colSpan="3" @tap="setRecipeFilter(item)" />
              <Label col="0" class="bx" :text="icon[selectedFilterType]" />
              <Label col="1" :text="`${item}` | L" />
              <Label class="recipeCount" :text="getRecipeCount(item)" col="2" />
            </GridLayout>
            <GridLayout v-if="selectedFilterType=='tag' && !tagsWithRecipes.length" columns="*" rows="*">
              <Label class="noTags" :text="'No tags available' | L" textWrap="true" />
            </GridLayout>
          </StackLayout>
        </ScrollView>
      </StackLayout>
      <StackLayout row="1">
        <StackLayout class="hr" margin="0 8 8"></StackLayout>
        <GridLayout rows="48" columns="auto, *" class="sd-item orkm" :class="{
              'selected-sd-item': currentComponent === 'MealPlanner',
            }">
          <MDRipple row="0" colSpan="3" @tap="navigateTo(MealPlanner, 'MealPlanner', true)" />
          <Label col="0" row="0" class="bx" :text="icon.calendar" />
          <Label col="2" row="0" :text="'Meal Planner' | L" />
        </GridLayout>
        <!-- <GridLayout rows="48" columns="auto, *" class="sd-item orkm" :class="{
              'selected-sd-item': currentComponent === 'MealPlanner',
            }">
          <MDRipple row="0" colSpan="3" @tap="navigateTo(MealPlanner, 'MealPlanner', true)" />
          <Label col="0" row="0" class="bx" :text="icon.cart" />
          <Label col="2" row="0" :text="'Shopping List' | L" />
        </GridLayout> -->
        <StackLayout class="hr" margin="8"></StackLayout>
        <GridLayout class="sd-item orkm" :class="{
              'selected-sd-item': currentComponent == item.title,
            }" v-for="(item, index) in bottommenu" :key="index" rows="48" columns="auto, *">
          <MDRipple colSpan="3" @tap="navigateTo(item.component, 'item.title', true)" />
          <Label class="bx" col="0" :text="icon[item.icon]" />
          <Label col="2" :text="`${item.title}` | L" />
        </GridLayout>
      </StackLayout>
    </GridLayout>
    <Frame ~mainContent id="main-frame">
      <EnRecipes ref="enrecipes" :filterFavourites="filterFavourites" :filterTrylater="filterTrylater" :selectedCuisine="selectedCuisine" :selectedCategory="selectedCategory" :selectedTag="selectedTag" :closeDrawer="closeDrawer"
        :hijackGlobalBackEvent="hijackGlobalBackEvent" :releaseGlobalBackEvent="releaseGlobalBackEvent" />
    </Frame>
  </RadSideDrawer>
</Page>
</template>

<script>
import {
  ApplicationSettings,
  AndroidApplication,
  Application,
  Device
}
from "@nativescript/core"
import Theme from "@nativescript/theme"
import * as Toast from "nativescript-toast"
import * as application from "tns-core-modules/application"
import {
  localize
}
from "@nativescript/localize"
import {
  mapActions,
  mapState
}
from "vuex"
import EnRecipes from "./EnRecipes"
import MealPlanner from "./MealPlanner"
import Settings from "./Settings"
import About from "./About"
import PromptDialog from "./modal/PromptDialog"
export default {
  data() {
    return {
      selectedCuisine: null,
      selectedCategory: null,
      selectedTag: null,
      selectedFilterType: 'cuisine',
      filterFavourites: false,
      filterTrylater: false,
      MealPlanner: MealPlanner,
      topmenu: [ {
        title: "EnRecipes",
        component: "EnRecipes",
        icon: "home",
      }, {
        title: "Try Later",
        component: "Try Later",
        icon: "trylater",
      }, {
        title: "Favourites",
        component: "Favourites",
        icon: "heart",
      }, ],
      bottommenu: [ {
        title: "Settings",
        component: Settings,
        icon: "cog",
      }, {
        title: "About",
        component: About,
        icon: "info",
      }, ],
      appTheme: "Light",
    }
  },
  components: {
    EnRecipes,
    MealPlanner,
  },
  computed: {
    ...mapState( [ "icon", "recipes", "cuisines", "categories", "yieldUnits", "mealPlans", "currentComponent" ] ),
    getCurrentPath() {
      let path = "/"
      if ( this.selectedCuisine ) path += localize( this.selectedCuisine )
      else path = "Cuisines"
      if ( this.selectedCategory ) path += "/" + localize( this.selectedCategory )
      if ( this.selectedTag ) path += "/" + localize( this.selectedTag )
      return path;
    },
    getRecipeList() {
      switch ( this.selectedFilterType ) {
        case 'cuisine':
          return this.cuisinesWithRecipes
          break;
        case 'category':
          return this.categoriesWithRecipes
          break;
        case 'tag':
          return this.tagsWithRecipes
          break;
      }
    },
    cuisinesWithRecipes() {
      let arr = this.recipes.map( ( e ) => e.cuisine )
      return arr.length ? [ "All Cuisines", ...new Set( arr ) ] : []
    },
    categoriesWithRecipes() {
      let arr = this.recipes.map( e => ( this.selectedCuisine === "All Cuisines" || e.cuisine === this.selectedCuisine ) && e.category ).filter( e => e )
      return arr.length ? [ "All Categories", ...new Set( arr ) ] : []
    },
    tagsWithRecipes() {
      let arr = this.recipes.map( e => {
        if ( this.selectedCuisine === "All Cuisines" && this.selectedCategory === "All Categories" && e.tags.length ) return e.tags;
        else if ( this.selectedCuisine === "All Cuisines" && e.category === this.selectedCategory && e.tags.length ) return e.tags;
        else if ( this.selectedCategory === "All Categories" && e.cuisine === this.selectedCuisine && e.tags.length ) return e.tags;
        else if ( e.category === this.selectedCategory && e.cuisine === this.selectedCuisine && e.tags.length ) return e.tags;
      } ).flat().filter( e => e )
      let showAllTags = this.selectedCuisine === "All Cuisines" && this.selectedCategory === "All Categories"
      return arr.length ? [ !showAllTags && "All Tags", ...new Set( arr ) ].filter( e => e ) : []
    },
  },
  methods: {
    ...mapActions( [ "setCurrentComponentAction", "initializeListItems", "initializeRecipes", "initializeMealPlans", "setShakeAction" ] ),
    onPageLoad() {
      if ( this.appTheme === "Light" ) {
        const View = android.view.View
        const window = Application.android.startActivity.getWindow()
        const decorView = window.getDecorView()
        decorView.setSystemUiVisibility( View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR )
      }
    },
    // HELPERS
    setRecipeFilter( item ) {
      this.$navigateBack( {
        frame: "main-frame",
        backstackVisible: false
      } )
      this.filterFavourites = this.filterTrylater = false
      if ( this.selectedCuisine == null ) {
        this.selectedFilterType = 'category'
        this.selectedCuisine = item
      } else if ( this.selectedCategory == null ) {
        this.selectedFilterType = 'tag'
        this.selectedCategory = item
        if ( !this.tagsWithRecipes.length ) this.closeDrawer()
      } else {
        this.selectedTag = item;
        this.closeDrawer()
      }
      this.setCurrentComponentAction( "Filtered result" )
      this.$refs.enrecipes.updateFilter()
    },
    previousRecipeFilter() {
      if ( this.selectedCategory ) {
        this.selectedFilterType = 'category'
        this.selectedTag = this.selectedCategory = null
        this.setCurrentComponentAction( "Filtered result" )
      } else {
        this.selectedFilterType = 'cuisine'
        this.selectedCuisine = null
        this.setCurrentComponentAction( "EnRecipes" )
      }
      this.$refs.enrecipes.updateFilter()
    },
    closeDrawer() {
      this.$refs.drawer.nativeView.closeDrawer()
    },
    getRecipeCount( arg ) {
      let count = ''
      switch ( arg ) {
        case 'EnRecipes':
          count = this.recipes.length
          break;
        case 'Try Later':
          count = this.recipes.filter( e => !e.tried ).length
          break;
        case 'Favourites':
          count = this.recipes.filter( e => e.isFavorite ).length
          break;
        default: {
          switch ( this.selectedFilterType ) {
            case 'cuisine':
              count = this.recipes.filter( e => arg === "All Cuisines" ? e.cuisine : e.cuisine === arg ).length
              break;
            case 'category':
              count = this.recipes.filter( e => this.selectedCuisine === "All Cuisines" ? arg === "All Categories" ? e.category : e.category === arg : arg === "All Categories" ? e.cuisine === this.selectedCuisine && e.category : e.cuisine === this
                .selectedCuisine && e.category === arg ).length
              break;
            case 'tag':
              count = this.recipes.filter( e => {
                if ( this.selectedCuisine === "All Cuisines" && this.selectedCategory === "All Categories" ) {
                  return e.tags.includes( arg ) || arg === "All Tags"
                } else if ( this.selectedCuisine === "All Cuisines" && e.category === this.selectedCategory ) {
                  return e.tags.includes( arg ) || arg === "All Tags"
                } else if ( this.selectedCategory === "All Categories" && e.cuisine === this.selectedCuisine ) {
                  return e.tags.includes( arg ) || arg === "All Tags"
                } else if ( e.category === this.selectedCategory && e.cuisine === this.selectedCuisine ) {
                  return e.tags.includes( arg ) || arg === "All Tags"
                }
              } ).length
              break;
          }
        }
      }
      return count
    },
    // NAVIGATION HANDLERS
    hijackGlobalBackEvent() {
      AndroidApplication.on( AndroidApplication.activityBackPressedEvent, this.globalBackEvent )
    },
    releaseGlobalBackEvent() {
      AndroidApplication.off( AndroidApplication.activityBackPressedEvent, this.globalBackEvent )
    },
    globalBackEvent( args ) {
      function preventDefault() {
        args.cancel = true
      }
      if ( this.$refs.drawer && this.$refs.drawer.nativeView.getIsOpen() ) {
        preventDefault()
        this.closeDrawer()
      } else if (
        [ "Favourites", "Try Later", "Filtered result" ].includes( this.currentComponent ) ) {
        preventDefault()
        this.setCurrentComponentAction( "EnRecipes" )
        this.filterFavourites = this.filterTrylater = false
        this.selectedTag = this.selectedCategory = this.selectedCuisine = null
        this.selectedFilterType = "cuisine"
        this.$refs.enrecipes.updateFilter()
        this.releaseGlobalBackEvent()
      }
    },
    navigateTo( to, title, isTrueComponent ) {
      if ( title !== this.currentComponent ) {
        if ( isTrueComponent ) {
          this.$navigateTo( to, {
            frame: "main-frame",
            backstackVisible: false
          } )
          this.closeDrawer()
        } else {
          this.releaseGlobalBackEvent()
          this.hijackGlobalBackEvent()
          this.setCurrentComponentAction( to )
          this.$navigateBack( {
            frame: "main-frame",
            backstackVisible: false
          } )
          this.filterFavourites = to === "Favourites" ? true : false
          this.filterTrylater = to === "Try Later" ? true : false
          this.$refs.enrecipes.updateFilter()
          this.closeDrawer()
        }
      } else {
        this.closeDrawer()
      }
      this.selectedTag = this.selectedCategory = this.selectedCuisine = null
      this.selectedFilterType = "cuisine"
    },
  },
  created() {
    this.appTheme = ApplicationSettings.getString( "appTheme", "Light" )
    setTimeout( ( e ) => {
      Theme.setMode( Theme[ this.appTheme ] )
    }, 10 )
    if ( !this.recipes.length ) this.initializeRecipes()
    this.initializeListItems()
    if ( !this.mealPlans.length ) this.initializeMealPlans()
    this.setShakeAction( ApplicationSettings.getBoolean( "shakeEnabled", true ) )
  },
}
</script>
