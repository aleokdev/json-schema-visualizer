<template>
    <div id="app" class="columns is-gapless" style="margin:0;padding:0">
        <textarea v-if="showEditor" v-model.lazy="rawSchema" @change="changeInput($event.target.value)" placeholder="enter schema"
                  class="column is-one-third" style="margin:0 20px;min-height:500px"/>
        <json-schema v-if="schema != null" :key="rawSchema" v-model="schema" class="column" :max-level="maxLevel" style="padding-left:10px"/>
        <!--<span v-else>-->
          <!--<span v-if="error != null" style="color:red">-->
            <!--Error parsing JSON: {{error}}-->
          <!--</span>-->
          <!--<pre v-else>-->
            <!--try-->

            <!--{ "$ref": "/example-schema.json" }-->

            <!--or-->


            <!--{-->
              <!--"$id": "https://example.com/person.schema.json",-->
              <!--"$schema": "http://json-schema.org/draft-07/schema#",-->
              <!--"title": "Person",-->
              <!--"type": "object",-->
              <!--"properties": {-->
                <!--"firstName": {-->
                  <!--"type": "string",-->
                  <!--"description": "The person's first name."-->
                <!--},-->
                <!--"lastName": {-->
                  <!--"type": "string",-->
                  <!--"description": "The person's last name."-->
                <!--},-->
                <!--"age": {-->
                  <!--"description": "Age in years which must be equal to or greater than zero.",-->
                  <!--"type": "integer",-->
                  <!--"minimum": 0-->
                <!--}-->
              <!--}-->
            <!--}-->
          <!--</pre>-->
        <!--</span>-->

      <!--<textarea v-model.lazy="data" @change="validateData" class="column"></textarea>-->
    </div>
</template>

<script>
  import JsonSchema from './components/JsonSchema.vue'
  import Ajv from 'ajv'
  import 'hint.css/hint.css'

  const urlParams = new URLSearchParams(window.location.search);

  const correctRules = (value) => {
    if (value == null) { return; }
    if (value["allOf"] !== undefined && value["allOf"].length === 1) {
      value["$ref"] = value["allOf"][0]["$ref"];
      value["allOf"] = undefined;
    }
    if (value["anyOf"] !== undefined) {
      value["anyOf"] = value["anyOf"].filter(item => item["type"] === undefined || item["type"] != "null");
    }
    if (value["anyOf"] !== undefined && value["anyOf"].length === 1) {
      value["$ref"] = value["anyOf"][0]["$ref"];
      value["anyOf"] = undefined;
    }
    for(const child of Object.values(value)) {
      if(typeof(child) === "object") {
        correctRules(child);
      }
    }
  }

  const parseRawInput = (value) => {
    try {
      const schema = JSON.parse(value);
      correctRules(schema);
      const error = null;
      return {schema, error}
    } catch (error) {
      return {error}
    }
  }

  export default {
    name: 'app',
    components: {
      JsonSchema
    },
    methods: {
      changeInput: function (value) {
        if (value == '') {
          this.error = this.schema = null
        } else {
          const {schema, error} = parseRawInput(value)
          this.schema = schema
          this.error = error
        }
      },
      validateData: function() {
        const ajv = new Ajv({allErrors: true, verbose: true, jsonPointers: true }); // options can be passed, e.g. {allErrors: true}
        const validate = ajv.compile(JSON.parse(this.rawSchema));
        const valid = validate(JSON.parse(this.data));
        if (!valid) {
          // eslint-disable-next-line
          console.log(validate.errors);
        }
      }
    },
    created: function() {
      var vue = this;
      if (urlParams.get('surl') != null) {
        fetch(decodeURIComponent(urlParams.get('surl'))).then(r => r.text()).then(t => {
          vue.rawSchema = t;
          const {schema, error} = parseRawInput(t);
          vue.schema = schema;
          vue.error = error;
        })
      }
    },
    data: function () {
      let rawSchema = urlParams.get('s') != null ? decodeURIComponent(urlParams.get('s')) : "{}";
      const {schema, error} = parseRawInput(rawSchema)

      const hideEditor = urlParams.get('hideEditor');
      const maxLevel = urlParams.get('maxLevel') != null ? parseInt(urlParams.get('maxLevel')) : 4;

      return {
        rawSchema: rawSchema || '',
        schema: schema,
        showEditor: hideEditor == null,
        maxLevel: maxLevel,
        error: error,
        // data: '{"name": "test", "from": {"database": "database", "schema": "schema"}}',
      }
    }
  }
</script>
