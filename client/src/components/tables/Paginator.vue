<style scope>
  .button-width {
    width: 50px;
  }

  .pagination-showing {
    padding-top: 10px;
    font-size: 14px;
    display: inline;
  }

  .label-nomorepages {
    padding-right:20px;
    color:gray;
    font: bold;
    font: italic;
  }

</style>

<template>
  <div class="row">
    <div class="col-sm-6">
      <!--Per Page-->
      <div align="left" class="pagination-showing">
        {{ ts['pages'] }}
        <select type="text" v-model="perPage" @change="getData(url)">
          <option v-for="item in perPagesList" :value="item">{{item}}</option>
        </select>
        <span> {{ ts['showing'] }}: {{pagination.from}} {{ ts['to'] }} {{pagination.to}} {{ ts['of'] }} {{pagination.total}} {{ ts['items'] }} </span>
      </div>
    </div>
    <div class="col-sm-6">
      <div align="right">
        <!--Pagination Buttons-->
        <a class="btn btn-sm btn-primary button-width"
        @click.prevent="setStartPageUrl"> {{ ts['start'] }} </a>
        <a class="btn btn-sm btn-primary button-width"
        @click.prevent="setPrevPageUrl"> {{ ts['prev'] }} </a>
        <a class="btn btn-sm btn-primary button-width"
        @click.prevent="setNextPageUrl"> {{ ts['next'] }} </a>
        <a class="btn btn-sm btn-primary button-width"
        @click.prevent="setEndPageUrl"> {{ ts['end'] }} </a>
        <br/> <br/>
      </div>
    </div>
    <!--No More Pages Label-->
    <div v-if="noMorePages" class="label-nomorepages" align="right">
      {{ ts['noMorePages'] }}
    </div>
  </div>
</template>

<script>
  // vuex store
  // import store from '../../store/Companies/Store';
  import store from '../../store/Store';
  // my components
  import MyLang from '../../components/languages/Languages';

  export default {

    mixins: [MyLang],

    props: ['url'],

    data() {
      return {
        noMorePages: false,
        perPagesList: ['5', '10', '25', '50', '100'],
        perPage: '10',
      };
    },

    mounted() {
      this.perPage = store.getters[`${this.$parent.moduleName}/getPerPage`];
    },

    computed: {
      pagination() {
        return store.getters[`${this.$parent.moduleName}/getPagination`];
      },
      searchText() {
        return store.getters[`${this.$parent.moduleName}/getSearchText`];
      },
      companySelected() {
        if (store.getters[`${this.$parent.moduleName}/getCompanySelected`] === undefined) {
          return null;
        }
        return store.getters[`${this.$parent.moduleName}/getCompanySelected`];
      },
      filterSelected() {
        return store.getters[`${this.$parent.moduleName}/getFilterSelected`];
      },
      fieldOrderBy() {
        return store.getters[`${this.$parent.moduleName}/getFieldOrderBy`];
      },
      orderBy() {
        return store.getters[`${this.$parent.moduleName}/getOrderBy`];
      },
      getUrlParams() {
        const urlParams = store.getters[`${this.$parent.moduleName}/getUrlParams`];
        return (urlParams !== null) ? (urlParams.search).substring(1) : '';
      },
    },

    methods: {
      setStartPageUrl() {
        store.dispatch(`${this.$parent.moduleName}/getUrlParams`);
        const pageUrl = `${this.pagination.path}?${this.getUrlParams}`;
        this.getData(pageUrl);
      },
      setPrevPageUrl() {
        store.dispatch(`${this.$parent.moduleName}/getUrlParams`);
        const prevUrl = `${this.pagination.prev_page_url}&${this.getUrlParams}`;
        const pageUrl = (this.pagination.prev_page_url) ? prevUrl : null;
        this.getData(pageUrl);
      },
      setNextPageUrl() {
        store.dispatch(`${this.$parent.moduleName}/getUrlParams`);
        const nextUrl = `${this.pagination.next_page_url}&${this.getUrlParams}`;
        const pageUrl = (this.pagination.next_page_url) ? nextUrl : null;
        this.getData(pageUrl);
      },
      setEndPageUrl() {
        store.dispatch(`${this.$parent.moduleName}/getUrlParams`);
        const pageUrl = `${this.pagination.path}?page=${this.pagination.last_page}&${this.getUrlParams}`;
        this.getData(pageUrl);
      },
      // getUrlParams() {
        // return `companyId=${this.companySelected}&
        //   searchText=${this.searchText}&
        //   filterSelected=${this.filterSelected}&
        //   itemsByPage=${this.perPage}&
        //   fieldOrderBy=${this.fieldOrderBy}&
        //   orderBy=${this.orderBy}`;
      // },
      getData(pageUrl) {
        store.commit(`${this.$parent.moduleName}/UPDATE_LOADING`, true);
        store.commit(`${this.$parent.moduleName}/UPDATE_PER_PAGE`, this.perPage);
        this.noMorePages = false;
        if (pageUrl !== null) {
          const newPageUrl = pageUrl || this.url;
          store.dispatch(`${this.$parent.moduleName}/getUrlParams`);
          console.log(`${this.$parent.moduleName}/getData`, `${newPageUrl}?${this.getUrlParams}`);
          store.dispatch(`${this.$parent.moduleName}/getData`, `${newPageUrl}?${this.getUrlParams}`)
          .then(() => {
            store.commit(`${this.$parent.moduleName}/UPDATE_LOADING`, false);
          })
          .catch(() => {
            store.commit(`${this.$parent.moduleName}/SHOW_MESSAGE`, this.getUnexpectedErrorMsg());
            store.commit(`${this.$parent.moduleName}/UPDATE_LOADING`, false);
          });
        } else {
          this.noMorePages = true;
          store.commit(`${this.$parent.moduleName}/UPDATE_LOADING`, false);
        }
      },
      getUnexpectedErrorMsg() {
        const response = { data: { error: '', message: '' } };
        response.data.message = 'Caught an unexpected error in method getData of the Pagination Component, contact the system administrator ';
        response.data.error = true;
        return response;
      },
    },
  };
</script>
