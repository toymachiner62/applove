<template>
  <div class="hello">
    <h1>This is YOU</h1>
    <a href="https://github.com/login/oauth/authorize?scope=user:email&client_id=ad8537ed99290f08e25b">Login</a>

    <div v-if="user">
      <img v-bind:src="user.avatar_url" />
      <div>
        <h2><a v-bind:href="user.html_url">{{user.name}}</a></h2>
        <h3>{{user.bio}}</h3>
      </div>

      <div>
        <h2>Highlights</h2>
        <ul>
          <li>{{user.followers}} followers</li>
          <li>Following {{user.following}}</li>
          <li>{{user.public_gists}} public gists</li>
          <li>{{user.public_repos}} public repos</li>
        </ul>
      </div>
    </div>

    <div class="repos">
      <div v-if="myRepos && myRepos.length > 0">
        <h3>My Repos</h3>
        <ul>
          <li v-for="repo in orderMyReposByStars" v-bind:key="repo.id">
            <div class="repo">
              {{repo.name}}
              <div class="pull-right">
                <i class="fas fa-star"></i>
                <span class="bold">{{repo.stargazers_count}}</span>
              </div>
            </div>
          </li>
        </ul>
      </div>

      <div v-if="collaboratorRepos && collaboratorRepos.length > 0">
        <h3>Repos I Collaborate On</h3>
        <ul>
          <li v-for="repo in orderCollaboratorReposByStars" v-bind:key="repo.id">
            <div class="repo">
              {{repo.name}}
              <div class="pull-right">
                <i class="fas fa-star"></i>
                <span class="bold">{{repo.stargazers_count}}</span>
              </div>
            </div>
          </li>
        </ul>
      </div>

      <div v-if="organizationRepos && organizationRepos.length > 0">
        <h3>Repos My Organization Has</h3>
        <ul>
          <li v-for="repo in orderOrganizationReposByStars" v-bind:key="repo.id">
            <div class="repo">
              {{repo.name}}
              <div class="pull-right">
                <i class="fas fa-star"></i>
                <span class="bold">{{repo.stargazers_count}}</span>
              </div>
            </div>
          </li>
        </ul>
      </div>

    </div>

  </div>
</template>

<script>
// import axios from 'axios'
import Octokit from '@octokit/rest'
const octokit = new Octokit()

export default {
  name: 'HelloWorld',
  data () {
    return {
      user: null,
      myRepos: null,
      collaboratorRepos: null,
      organizationRepos: null
    }
  },
  computed: {
    orderMyReposByStars: function () {
      // Clone myRepos so that we're not mutating the original, sort it by stars asc, then reverse it to get sorted by stars desc
      return this.myRepos.slice(0).sort((a, b) => a.stargazers_count > b.stargazers_count).reverse()
    },
    orderCollaboratorReposByStars: function () {
      // Clone collaboratorRepos so that we're not mutating the original, sort it by stars asc, then reverse it to get sorted by stars desc
      return this.collaboratorRepos.slice(0).sort((a, b) => a.stargazers_count > b.stargazers_count).reverse()
    },
    orderOrganizationReposByStars: function () {
      // Clone organizationRepos so that we're not mutating the original, sort it by stars asc, then reverse it to get sorted by stars desc
      return this.organizationRepos.slice(0).sort((a, b) => a.stargazers_count > b.stargazers_count).reverse()
    }
  },
  mounted: function () {
    let token = this.$route.query.access_token
    getUserInfo(token)
      .then(user => {
        this.user = user
      })
      .catch(e => {
        console.error(e)
      })
    getUserRepos(token, 'owner')
      .then(repos => {
        this.myRepos = repos
        // console.log(this.myRepos)
      })
      .catch(e => {
        console.error(e)
      })
    getUserRepos(token, 'collaborator')
      .then(repos => {
        this.collaboratorRepos = repos
        console.log(this.collaboratorRepos)
      })
      .catch(e => {
        console.error(e)
      })
    getUserRepos(token, 'organization_member')
      .then(repos => {
        this.organizationRepos = repos
        // console.log(this.organizationRepos)
      })
      .catch(e => {
        console.error(e)
      })
  }
}

// Pagination straight from the github octokit api https://github.com/octokit/rest.js#pagination
async function paginate (method) {
  let response = await method({per_page: 100})
  let {data} = response
  while (octokit.hasNextPage(response)) {
    response = await octokit.getNextPage(response)
    data = data.concat(response.data)
  }
  console.log('data = ', data)
  return data
}

/**
 * Gets the info for the authenticated user
 *
 * @param token - The auth token
 */
function getUserInfo (token) {
  return new Promise((resolve, reject) => {
    octokit.authenticate({
      type: 'oauth',
      token: token
    })

    octokit.users.get({})
      .then(res => {
        return resolve(res.data)
      })
      .catch(e => {
        return reject(e)
      })
  })
}

/**
 * Gets the repos for the authenticated user
 *
 * @param token - The auth token
 * @param affiliation - The afilliation to the repo
 */
function getUserRepos (token, affiliation) {
  return new Promise((resolve, reject) => {
    octokit.authenticate({
      type: 'oauth',
      token: token
    })

    paginate(() => octokit.repos.getAll({affiliation: affiliation}))
      .then(repos => {
        return resolve(repos)
      })
      .catch(e => {
        return reject(e)
      })
  })
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  clear: both;
}
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.repos li {
  display: block;
}

.repo {
  text-align: left;
  border: solid 1px #999999;
  margin: 10px;
  padding: 10px;
}

.repos .fa-star {
  color: #f4d941;
}

.bold {
  font-weight: bold;
}
</style>
