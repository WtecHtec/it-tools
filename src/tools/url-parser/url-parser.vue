<script setup lang="ts">
import InputCopyable from '../../components/InputCopyable.vue';
import { isNotThrowing } from '@/utils/boolean';
import { withDefaultOnError } from '@/utils/defaults';
import JSON5 from 'json5';
import type { UseValidationRule } from '@/composable/validation';


const urlToParse = ref('https://me:pwd@it-tools.tech:3000/url-parser?key1=value&key2=value2#the-hash');

const urlParsed = computed(() => withDefaultOnError(() => new URL(urlToParse.value), undefined));
const urlValidationRules = [
  {
    validator: (value: string) => isNotThrowing(() => new URL(value)),
    message: 'Invalid url',
  },
];

const properties: { title: string; key: keyof URL }[] = [
  { title: 'Protocol', key: 'protocol' },
  { title: 'Username', key: 'username' },
  { title: 'Password', key: 'password' },
  { title: 'Hostname', key: 'hostname' },
  { title: 'Port', key: 'port' },
  { title: 'Path', key: 'pathname' },
  { title: 'Params', key: 'search' },
];

const defaultValue = '{\n\t"hello": [\n\t\t"world"\n\t]\n}';
const transformer = (value: string) => withDefaultOnError(() => JSON.stringify(JSON5.parse(value), null, 0), '');

const defaultValueFomr = (): string => withDefaultOnError(() => {
	
	const value = {}
	console.log('urlParsed---', (urlParsed as any)?.searchParams)
	// for(let [k, v] in Object.entries(Object.fromEntries(urlParsed?.searchParams.entries() ?? []))) {
	// 	value[k] = v
	// }
	return JSON.stringify(value, null, 0)
}, '');
const output = computed(() => {
	// console.log('urlParsed---', (urlParsed.value?.searchParams))
	// const value:  = {}
	const rawJson :{ [key: string]: any }= {}
		for(let [k, v] of Object.entries(Object.fromEntries(urlParsed.value?.searchParams.entries() ?? []))) {
			rawJson[k] = v
	}
	// console.log(value)
	// return transformer(JSON.stringify(value))
	return JSON.stringify(rawJson, null, 2)
});

const rules: UseValidationRule<string>[] = [
  {
    validator: (v: string) => v === '' || JSON5.parse(v),
    message: 'Provided JSON is not valid.',
  },
];
</script>

<template>
  <c-card>
    <c-input-text
      v-model:value="urlToParse"
      label="URL链接:"
      placeholder="URL链接."
      raw-text
      :validation-rules="urlValidationRules"
    />

    <n-divider />

    <InputCopyable
      v-for="{ title, key } in properties"
      :key="key"
      :label="title"
      :value="(urlParsed?.[key] as string) ?? ''"
      readonly
      label-position="left"
      label-width="110px"
      mb-2
      placeholder=" "
    />

    <div
      v-for="[k, v] in Object.entries(Object.fromEntries(urlParsed?.searchParams.entries() ?? []))"
      :key="k"
      mb-2
      w-full
      flex
    >
      <div style="flex: 1 0 110px">
        <icon-mdi-arrow-right-bottom />
      </div>

      <InputCopyable :value="k" readonly />
      <InputCopyable :value="v" readonly />
    </div>  
		<div mb-5px>
     参数json
    </div>
		<textarea-copyable :value="output"  language="json"/>
  </c-card>
</template>

<style lang="less" scoped>
.n-input-group-label {
  text-align: right;
}
.n-input-group {
  margin: 2px 0;
}
</style>
