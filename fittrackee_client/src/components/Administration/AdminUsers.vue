<template>
  <div id="admin-users" class="admin-card">
    <Card>
      <template #title>{{ capitalize($t('admin.USER', 0)) }}</template>
      <template #content>
        <button class="top-button" @click.prevent="$router.push('/admin')">
          {{ $t('admin.BACK_TO_ADMIN') }}
        </button>
        <UsersNameFilter @filterOnUsername="searchUsers" />
        <FilterSelects
          :sort="sortList"
          :order_by="orderByList"
          :query="query"
          message="admin.USERS.SELECTS.ORDER_BY"
          @updateSelect="reloadUsers"
        />
        <div class="no-users" v-if="users.length === 0">
          {{ $t('user.NO_USERS_FOUND') }}
        </div>
        <div class="responsive-table" v-else>
          <table>
            <thead>
              <tr>
                <th>#</th>
                <th class="left-text">{{ $t('user.USERNAME') }}</th>
                <th class="left-text">{{ $t('user.EMAIL') }}</th>
                <th class="left-text">
                  {{ $t('user.PROFILE.REGISTRATION_DATE') }}
                </th>
                <th>
                  {{ capitalize($t('workouts.WORKOUT', 0)) }}
                </th>
                <th>{{ $t('admin.ACTIVE') }}</th>
                <th>{{ $t('user.ADMIN') }}</th>
                <th>{{ $t('admin.ACTION') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="user in users" :key="user.username">
                <td>
                  <span class="cell-heading">
                    {{ $t('user.PROFILE.PICTURE') }}
                  </span>
                  <UserPicture :user="user" />
                </td>
                <td>
                  <span class="cell-heading">
                    {{ $t('user.USERNAME') }}
                  </span>
                  <router-link :to="`/admin/users/${user.username}`">
                    {{ user.username }}
                  </router-link>
                </td>
                <td>
                  <span class="cell-heading">
                    {{ $t('user.EMAIL') }}
                  </span>
                  {{ user.email }}
                </td>
                <td>
                  <span class="cell-heading">
                    {{ $t('user.PROFILE.REGISTRATION_DATE') }}
                  </span>
                  {{
                    formatDate(
                      user.created_at,
                      authUser.timezone,
                      authUser.date_format
                    )
                  }}
                </td>
                <td class="text-center">
                  <span class="cell-heading">
                    {{ capitalize($t('workouts.WORKOUT', 0)) }}
                  </span>
                  {{ user.nb_workouts }}
                </td>
                <td class="text-center">
                  <span class="cell-heading">
                    {{ $t('admin.ACTIVE') }}
                  </span>
                  <i
                    :class="`fa fa${user.is_active ? '-check' : ''}-square-o`"
                    aria-hidden="true"
                  />
                </td>
                <td class="text-center">
                  <span class="cell-heading">
                    {{ $t('user.ADMIN') }}
                  </span>
                  <i
                    :class="`fa fa${user.admin ? '-check' : ''}-square-o`"
                    aria-hidden="true"
                  />
                </td>
                <td class="text-center">
                  <span class="cell-heading">
                    {{ $t('admin.ACTION') }}
                  </span>
                  <button
                    :class="{ danger: user.admin }"
                    :disabled="user.username === authUser.username"
                    @click="updateUser(user.username, !user.admin)"
                  >
                    {{
                      $t(
                        `admin.USERS.TABLE.${
                          user.admin ? 'REMOVE' : 'ADD'
                        }_ADMIN_RIGHTS`
                      )
                    }}
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
          <Pagination
            v-if="pagination.page"
            path="/admin/users"
            :pagination="pagination"
            :query="query"
          />
          <ErrorMessage :message="errorMessages" v-if="errorMessages" />
          <button @click.prevent="$router.push('/admin')">
            {{ $t('admin.BACK_TO_ADMIN') }}
          </button>
        </div>
      </template>
    </Card>
  </div>
</template>

<script setup lang="ts">
  import {
    ComputedRef,
    Ref,
    computed,
    reactive,
    watch,
    capitalize,
    onBeforeMount,
    onUnmounted,
  } from 'vue'
  import { LocationQuery, useRoute, useRouter } from 'vue-router'

  import FilterSelects from '@/components/Common/FilterSelects.vue'
  import Pagination from '@/components/Common/Pagination.vue'
  import UserPicture from '@/components/User/UserPicture.vue'
  import UsersNameFilter from '@/components/Users/UsersNameFilter.vue'
  import { AUTH_USER_STORE, ROOT_STORE, USERS_STORE } from '@/store/constants'
  import { IPagination, TPaginationPayload } from '@/types/api'
  import { IAuthUserProfile, IUserProfile } from '@/types/user'
  import { useStore } from '@/use/useStore'
  import { getQuery, sortList } from '@/utils/api'
  import { formatDate } from '@/utils/dates'

  const store = useStore()
  const route = useRoute()
  const router = useRouter()

  const orderByList: string[] = [
    'is_active',
    'admin',
    'created_at',
    'username',
    'workouts_count',
  ]
  const defaultOrderBy = 'created_at'
  let query: TPaginationPayload = reactive(
    getQuery(route.query, orderByList, defaultOrderBy)
  )
  const authUser: ComputedRef<IAuthUserProfile> = computed(
    () => store.getters[AUTH_USER_STORE.GETTERS.AUTH_USER_PROFILE]
  )
  const users: ComputedRef<IUserProfile[]> = computed(
    () => store.getters[USERS_STORE.GETTERS.USERS]
  )
  const pagination: ComputedRef<IPagination> = computed(
    () => store.getters[USERS_STORE.GETTERS.USERS_PAGINATION]
  )
  const errorMessages: ComputedRef<string | string[] | null> = computed(
    () => store.getters[ROOT_STORE.GETTERS.ERROR_MESSAGES]
  )

  onBeforeMount(() => loadUsers(query))

  function loadUsers(queryParams: TPaginationPayload) {
    store.dispatch(USERS_STORE.ACTIONS.GET_USERS, queryParams)
  }
  function searchUsers(username: Ref<string>) {
    reloadUsers('q', username.value)
  }

  function updateUser(username: string, admin: boolean) {
    store.dispatch(USERS_STORE.ACTIONS.UPDATE_USER, {
      username,
      admin,
    })
  }
  function reloadUsers(queryParam: string, queryValue: string) {
    query[queryParam] = queryValue
    if (queryParam === 'per_page') {
      query.page = 1
    }
    router.push({ path: '/admin/users', query })
  }

  onUnmounted(() => {
    store.dispatch(USERS_STORE.ACTIONS.EMPTY_USERS)
  })

  watch(
    () => route.query,
    (newQuery: LocationQuery) => {
      query = getQuery(newQuery, orderByList, defaultOrderBy, { query })
      loadUsers(query)
    }
  )
</script>

<style lang="scss" scoped>
  @import '~@/scss/vars.scss';
  #admin-users {
    .top-button {
      display: none;
    }

    .no-users {
      display: flex;
      justify-content: center;
      padding: $default-padding * 2 0;
      font-weight: bold;
    }

    table {
      td {
        font-size: 1.1em;
      }
    }
    .left-text {
      text-align: left;
    }
    ::v-deep(.user-picture) {
      img {
        height: 30px;
        width: 30px;
      }
      .no-picture {
        font-size: 2em;
      }
    }

    @media screen and (max-width: $small-limit) {
      .top-button {
        display: block;
        margin-bottom: $default-margin * 2;
      }
      .pagination-center {
        margin-top: -3 * $default-margin;
      }
    }
  }
</style>
