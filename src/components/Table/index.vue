<template>
  <div class="flex flex-col items-center">
    <el-card class="w-full width-100">
      <div class="mb-5">
        <div class="flex items-center justify-between">
          <span class="text-gray-700 text-2xl text-left font-bold py-4">
            {{ name }}
          </span>

          <router-link
            :to="{ name: links.store.name }"
            class="el-button el-button--primary el-button--medium"
            v-if="links.store"
          >
            <i class="el-icon-plus"></i> Қўшиш
          </router-link>
        </div>
      </div>
      <div class="mb-0 parent-table">
        <table
          class="table-auto table-main"
          v-loading="loadingData"
          element-loading-text="Юкланмоқда.."
          element-loading-spinner="el-icon-loading"
        >
          <thead>
            <tr class="text-gray-700 text-xs text-center font-bold bg-gray-200">
              <th
                v-for="(column, index) in columns"
                :key="index"
                class="border px-4 py-2"
              >
                {{ column.title }}
              </th>

              <th class="border px-4 py-2">
                Амаллар
              </th>
            </tr>
          </thead>

          <thead>
            <tr class="text-gray-700 text-xs text-center font-bold bg-white">
              <th class="border px-4 py-2">
                #
              </th>

              <th
                v-for="(item, index) in filter"
                :key="index"
                v-show="!hasType(item.type, 'invisible')"
                class="border px-4 py-2"
              >
                <!-- if filter type will be input -->
                <template v-if="hasType(item.type, 'input')">
                  <el-input
                    style="min-width: 90px"
                    v-model="filter[index].value"
                    size="mini"
                    class="w-full"
                    :placeholder="item.name"
                  ></el-input>
                </template>
                <!-- end filter type input-->

                <!-- if filter type will be select  -->
                <template v-else-if="hasType(item.type, 'select')">
                  <el-select
                    style="min-width: 100px"
                    v-model="filter[index].value"
                    size="mini"
                    class="w-full"
                  >
                    <el-option label="Барчаси" :value="null"></el-option>
                    <el-option
                      v-for="(option, index) in item.options"
                      :key="index"
                      :label="option.name"
                      :value="option.id"
                    ></el-option>
                  </el-select>
                </template>
                <!-- end filter type select -->
                <!-- filter type date -->
                <template v-else-if="hasType(item.type, 'date')">
                  <el-date-picker
                    size="mini"
                    v-model="filter[index].value"
                    type="date"
                    value-format="yyyy-MM-dd"
                    :placeholder="filter[index].name"
                  >
                  </el-date-picker>
                </template>
                <!-- end filter type date -->
                <!-- none filter argument -->
                <template v-else-if="hasType(item.type, 'none')">
                  <!-- {{item.name}} -->
                </template>
                <!-- end none filter argument -->
              </th>

              <th class="border px-4 py-2">
                <el-button type="primary" size="mini" @click="updateFilter">
                  Қидириш
                </el-button>
              </th>
            </tr>
          </thead>

          <tbody v-if="data.length > 0">
            <tr
              style="height:50px"
              v-for="(item, indexMain) in data"
              :key="indexMain + 'ii'"
              class="text-gray-700 text-xs text-center font-normal"
              :class="getBGColor(indexMain)"
            >
              <td
                v-for="(value, name, index) in item"
                :key="index"
                class="border px-4 py-2"
              >
                <template v-if="hasType(name, 'id')">
                  {{ indexMain + 1 }}
                </template>

                <template v-else-if="hasType(name, 'name') && links.show">
                  <router-link
                    v-if="links.show.key == 'id'"
                    :to="{ name: links.show.name, params: { id: item.id } }"
                    class="text-blue-500 hover:text-blue-600"
                  >
                    {{ value }}
                  </router-link>
                  <router-link
                    v-else-if="links.show.key == 'tin'"
                    :to="{ name: links.show.name, params: { tin: item.tin } }"
                    class="text-blue-500 hover:text-blue-600"
                  >
                    {{ value }}
                  </router-link>
                </template>
                <template
                  v-else-if="
                    hasType(name, 'media') ||
                      hasType(name, 'order') ||
                      hasType(name, 'kpi')
                  "
                >
                  <el-tooltip
                    v-if="value && value.url"
                    class="item"
                    effect="dark"
                    content="Юклаб олиш"
                    placement="top-start"
                  >
                    <a
                      :href="value.url"
                      target="_blank"
                      class="el-button el-button--primary el-button--mini"
                    >
                      <i class="el-icon-download"></i>
                    </a>
                  </el-tooltip>
                  <el-alert
                    v-else
                    title="Файл топилмади"
                    type="error"
                    show-icon
                    :closable="false"
                  >
                  </el-alert>
                </template>
                <template v-else-if="hasType(name, 'type_id')">
                  {{ getRelation_name(item) }}
                </template>
                <template v-else>
                  {{ value }}
                </template>
              </td>
              <td class="border px-4 py-2">
                <el-button
                  v-if="links.delete"
                  type="danger"
                  size="mini"
                  icon="el-icon-delete"
                  @click="deleteItem(item.id)"
                ></el-button>

                <!-- show -->
                <router-link
                  v-if="links.show && links.show.key == 'id'"
                  :to="{ name: links.show.name, params: { id: item.id } }"
                  class="el-button el-button--info el-button--small"
                >
                  <i class="el-icon-view"></i>
                </router-link>
                <router-link
                  v-else-if="links.show && links.show.key == 'tin'"
                  :to="{ name: links.show.name, params: { tin: item.tin } }"
                  class="tel-button el-button--info el-button--small"
                >
                  <i class="el-icon-view"></i>
                </router-link>
                <!-- end show -->
                <!-- update -->
                <router-link
                  v-if="links.update && links.update.key == 'id'"
                  :to="{ name: links.update.name, params: { id: item.id } }"
                  class="el-button el-button--primary el-button--small"
                >
                  <i class="el-icon-edit"></i>
                </router-link>
                <router-link
                  v-else-if="links.update && links.update.key == 'tin'"
                  :to="{ name: links.update.name, params: { tin: item.tin } }"
                  class="el-button el-button--primary el-button--small"
                >
                  <i class="el-icon-edit"></i>
                </router-link>
                <!-- end update -->
              </td>
            </tr>
          </tbody>
          <div v-else class="center--item-postion py-2">
            <span>Маълумот йўқ.</span>
          </div>
        </table>
        <br />
        <pagination-index
          :pagination="pagination"
          @c-change="updatePaginationFunction"
        ></pagination-index>
      </div>
    </el-card>
  </div>
</template>

<script>
import list from '@/mixins/model/list';
import PaginationIndex from '@/components/Includes/Pagination';
import { mapGetters, mapActions } from 'vuex';
export default {
  mixins: [list],
  components: {
    PaginationIndex,
  },
  props: {
    pagination: {
      type: Object,
      default: () => {},
      required: true,
    },
    name: {
      type: String,
      default: () => null,
      required: false,
    },

    columns: {
      type: Object,
      default: () => {},
      required: true,
    },
    data: {
      type: Array,
      default: () => [],
      required: true,
    },
    filter: {
      type: Array,
      default: () => [],
      required: true,
    },
    model_name: {
      type: String,
      default: () => null,
      required: true,
    },
    model_name: {
      type: String,
      default: () => null,
    },
    action_name: {
      type: String,
      default: () => 'list',
    },
    links: {
      type: Object,
      default: () => {},
      required: true,
    },
  },
  data() {
    return {};
  },
  computed: {},
  created() {},
  methods: {
    ...mapActions({}),
  },
};
</script>
<style scoped>
.el-card__body a:hover {
  text-decoration: underline;
}
</style>
