<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png" />
    <w-button class="text-right mt6" @click="get_code">코드 요청</w-button>
    <w-button class="text-center mt6" @click="get_token(updateParam().get('code'))">토큰 요청</w-button>
    <HelloWorld msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";

export default {
  name: "Home",
  components: {
    HelloWorld
  },
  data() {
    return {
      shopId: 'tjagksro',
      authCodeParams: {
        response_type: 'code',
        client_id: 'f31zyAgabCWXPLDAqtLYdD',
        state: 'TEMP_CSRF_TOKEN',
        redirect_uri: 'https://widget-admin.netlify.app/',
        // Syncronize With: https://developers.cafe24.com/admin/apps/front/detail?client_id=f31zyAgabCWXPLDAqtLYdD
        scope: 'mall.read_application,mall.write_application,mall.read_category,mall.read_product,mall.write_product,mall.read_personal',
      },
      authToken: {
        headers: {
          // Basic {base64_encode({client_id}:{client_Secret})}   From Cafe24 Develope Account'
          Authorization: 'Basic ZjMxenlBZ2FiQ1dYUExEQXF0TFlkRDoyOWVCb01zZW9rS3ZPQWxoMWpXcVFN',
          'Content-Type': 'application/x-www-form-urlencoded',
        },
        data: {
          grant_type: "authorization_code",
          code: null, // fill as authCode
          redirect_uri: 'https://widget-admin.netlify.app/'
        }
      }
    }
  },
  mounted() {
    const param = this.updateParam()
    console.log(
      "$route.params: ", this.$route.params,
      "URLSearchParams", param,
    )
    if (!param.has('code')) {
      this.get_code()
    } else {
      this.get_token({ code: param.get('code') })
        .then((res) => console.log("Get Token Response: ", res))
    }
  },
  methods: {
    updateParam() {
      const paramOfUrl = window.location.search
      return new URLSearchParams(paramOfUrl)
    },
    makeParam (baseUrl='', params={'key': 'val'}) {
      baseUrl += '?'
      let combinedUrl = Object.entries(params).reduce((url, [key, val]) => {
        url += `${key}=${val}&`
        return url
      }, baseUrl)
      return combinedUrl.slice(0, combinedUrl.length - 1) // exclude last &
    },
    get_code() {
      let url = `https://${this.shopId}.cafe24api.com/api/v2/oauth/authorize`
      alert(this.makeParam(url, this.authCodeParams))
      // window.location.href = this.makeParam(url, this.authCodeParams)
    },
    get_token({ code }) {
      const url = 'https://tjagksro.cafe24api.com/api/v2/oauth/token'
      const t = this.authToken
      t.headers.Host = `${this.shopId}.cafe24api.com`
      t.headers.Referer = `${this.shopId}.cafe24api.com`
      t.headers.Origin = `${this.shopId}.cafe24api.com`
      t.headers['Access-Control-Allow-Origin'] = '*'
      const headers = t.headers
      console.log("Headers: ", headers)
      t.data.code = code
      const formData = new URLSearchParams();

      for (const [key, value] of Object.entries(t.data)) {
        formData.append(key, value)
      }

      return this.axios.post(url, formData, {
        headers,
        crossDomain: true,
      })
    },
  }
};
</script>
