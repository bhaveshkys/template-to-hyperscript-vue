<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:mx-auto w-full px-4 max-w-6xl">
      <div class="absolute inset-0 bg-gradient-to-r from-cyan-400 to-light-blue-500 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <div class="mx-auto">
          <div class="divide-y divide-gray-200">
            <div class="py-8 text-base leading-6 space-y-4 text-gray-700 sm:text-lg sm:leading-7">
              <h2 class="text-3xl font-extrabold text-gray-900 sm:text-4xl text-center">Vue Template to h() Converter</h2>
              <p class="text-gray-500 text-center">Enter Vue template syntax, props, and see the converted h() syntax.</p>
            </div>
            <div class="pt-6 space-y-4">
              <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div>
                  <label for="template" class="block text-sm font-medium text-gray-700 mb-2">Template Syntax</label>
                  <textarea
                    id="template"
                    v-model="templateSyntax"
                    rows="10"
                    class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                    placeholder="<div class='flex mb-4'>
  <div class='bg-gray-700 p-4 text-white'>
    <component :is='icon' :height='35' :width='35' />
  </div>
  <div class='flex flex-col justify-center ml-4 gap-y-2'>
    <div class='font-plusJakartaSans font-semibold'>{{ title }}</div>
    <div class='font-manrope text-sm'>{{ content }}</div>
  </div>
</div>"
                  ></textarea>
                </div>
                <div>
                  <label for="props" class="block text-sm font-medium text-gray-700 mb-2">Props</label>
                  <textarea
                    id="props"
                    v-model="propsInput"
                    rows="10"
                    class="shadow-sm focus:ring-indigo-500 focus:border-indigo-500 block w-full sm:text-sm border-gray-300 rounded-md"
                    placeholder="icon: {
  type: Object,
  required: true
},
title: {
  type: String,
  required: true
},
content: {
  type: String,
  required: true
}"
                  ></textarea>
                </div>
              </div>
              <div>
                <label for="h-syntax" class="block text-sm font-medium text-gray-700 mb-2">h() Syntax</label>
                <pre
                  id="h-syntax"
                  class="shadow-sm block w-full sm:text-sm border-gray-300 rounded-md bg-gray-50 p-4 overflow-auto"
                  style="height: 30rem;"
                >{{ convertedComponent }}</pre>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const templateSyntax = ref('')
const propsInput = ref('')

const convertedTemplate = computed(() => {
  // This is a simplified conversion. A full parser would be more complex.
  let converted = templateSyntax.value
    .replace(/<template>/g, '')
    .replace(/<\/template>/g, '')
    .replace(/<(\w+)([^>]*)>/g, (_, tag, attrs) => {
      const parsedAttrs = attrs.match(/(\w+)(?:=['"]([^'"]*)?['"])?/g) || []
      const attrObj = parsedAttrs.map(attr => {
        const [key, value] = attr.split('=')
        if (value === undefined) return `${key}: true`
        if (value.startsWith("'") || value.startsWith('"')) return `${key}: ${value}`
        return `${key}: ${value.replace(/'/g, '')}`
      }).join(', ')
      return `h('${tag}', { ${attrObj} }, [`
    })
    .replace(/<\/(\w+)>/g, '])')
    .replace(/{{([^}]+)}}/g, (_, exp) => `${exp.trim()}`)
    .replace(/:is='([^']+)'/g, (_, value) => `is: ${value}`)

  return `return () => [\n    ${converted}\n  ]`
})

const convertedProps = computed(() => {
  try {
    // Remove any trailing commas and wrap in curly braces to make it valid JSON
    const jsonProps = `{${propsInput.value.replace(/,\s*$/, '')}}`
    const props = Function(`return ${jsonProps}`)()
    return Object.entries(props).map(([key, value]) => {
      let propString = `    ${key}: {\n      type: ${value.type.name}`
      if (value.required) propString += `,\n      required: ${value.required}`
      if (value.default !== undefined) propString += `,\n      default: ${JSON.stringify(value.default)}`
      propString += '\n    }'
      return propString
    }).join(',\n')
  } catch (e) {
    return '    // Invalid props input'
  }
})

const convertedComponent = computed(() => {
  return `const MyComponent = {
  props: {
${convertedProps.value}
  },
  setup(props) {
    ${convertedTemplate.value}
  }
};`
})
</script>