<template>
  <div class="flex flex-col items-center w-full h-full justify-center">
    <div class="flex flex-row items-center gap-2 w-full justify-center">
      
      <!-- Left: Unstructured Data -->
      <div class="flex flex-col items-center">
        <div class="flex flex-row items-center relative">
          <!-- Document Icon representation -->
          <div class="absolute -left-3 top-2 w-8 h-12 border border-black bg-white"></div>
          <div class="absolute -left-1 top-1 w-8 h-12 border border-black bg-white"></div>
          
          <div class="border border-gray-400 p-1 bg-white z-10 w-24 shadow-sm flex flex-col text-[8px] font-mono text-black">
            <span v-for="i in 7" :key="i">TEXT[...]</span>
          </div>
        </div>
        <div class="mt-3 text-center text-[10px] font-semibold">
          <p>GB - TB - PB</p>
          <p class="font-normal">of unstructured data</p>
        </div>
      </div>

      <!-- Filter -->
      <div class="bg-green-300 border border-green-500 text-green-900 py-6 px-1 flex items-center justify-center shadow-sm">
        <span class=" whitespace-nowrap text-[10px] font-semibold tracking-wider">Data Quality Filter</span>
      </div>

      <!-- Arrow & Filtered Data -->
      <div class="flex items-center gap-1 relative">
        <div class="w-4 h-[1px] bg-gray-400"></div>
        <div class="flex flex-col items-center relative">
          <div class="absolute -top-10 flex flex-col items-center">
            <span class="text-[9px] text-center w-16 leading-tight">1-3% of<br>original tokens</span>
            <div class="text-gray-500 text-[10px] mt-1">⬇</div>
          </div>
          <div class="border border-gray-400 p-2 bg-white text-black shadow-sm font-mono text-[8px]">
            TEXT[...]
          </div>
        </div>
        <div class="w-4 h-[1px] bg-gray-400"></div>
        <div class="text-gray-400 -ml-2 text-[10px]">▶</div>
      </div>

      <!-- Middle: Model & GPUs -->
      <div class="flex flex-col items-center relative mx-2">
        <span class="text-purple-400 font-bold mb-1 text-[11px]">Model</span>
        <div class="w-24 h-24 rounded-full bg-[#bd8bfa] border border-[#a268ed] flex items-center justify-center text-lg text-black font-semibold shadow-md">
          LLM
        </div>
        <!-- GPUs -->
        <div class="flex gap-2 mt-3">
          <div class="border border-black p-1 text-[8px] font-bold relative flex items-center justify-center w-8 h-8 text-black bg-white">
            <span class="absolute -left-[3px] flex flex-col gap-[2px]"><div class="w-1 h-[1px] bg-black" v-for="i in 3" :key="i"></div></span>
            GPU
            <span class="absolute -right-[3px] flex flex-col gap-[2px]"><div class="w-1 h-[1px] bg-black" v-for="i in 3" :key="i"></div></span>
          </div>
          <div class="border border-black p-1 text-[8px] font-bold relative flex items-center justify-center w-8 h-8 text-black bg-white">
             <span class="absolute -left-[3px] flex flex-col gap-[2px]"><div class="w-1 h-[1px] bg-black" v-for="i in 3" :key="i"></div></span>
            GPU
            <span class="absolute -right-[3px] flex flex-col gap-[2px]"><div class="w-1 h-[1px] bg-black" v-for="i in 3" :key="i"></div></span>
          </div>
        </div>
      </div>

      <!-- Right: Vocabulary Table -->
      <div class="flex flex-col items-center ml-2">
        <table class="border-collapse border border-gray-300 text-[9px] bg-white text-black shadow-sm w-60">
          <thead>
            <tr class="bg-blue-200 text-left">
              <th class="border border-gray-300 p-1 font-semibold">Token String</th>
              <th class="border border-gray-300 p-1 font-semibold">Token ID</th>
              <th class="border border-gray-300 p-1 font-semibold">Embedding /<br>Vector Representation</th>
            </tr>
          </thead>
          <tbody class="font-mono">
            <tr>
              <td class="border border-gray-300 p-1">'_The'</td>
              <td class="border border-gray-300 p-1">37</td>
              <td class="border border-gray-300 p-1 tracking-tighter">[-0.0513, -0.0584, 0.0230, ...]</td>
            </tr>
            <tr>
              <td class="border border-gray-300 p-1">'_teacher'</td>
              <td class="border border-gray-300 p-1">3145</td>
              <td class="border border-gray-300 p-1 tracking-tighter">[-0.0335, 0.0167, 0.0484, ...]</td>
            </tr>
            <tr>
              <td class="border border-gray-300 p-1">'_teaches'</td>
              <td class="border border-gray-300 p-1">11749</td>
              <td class="border border-gray-300 p-1 tracking-tighter">[-0.0151, -0.0516, 0.0309, ...]</td>
            </tr>
            <tr>
              <td class="border border-gray-300 p-1">'_the'</td>
              <td class="border border-gray-300 p-1">8</td>
              <td class="border border-gray-300 p-1 tracking-tighter">[-0.0498, -0.0428, 0.0275, ...]</td>
            </tr>
            <tr>
              <td class="border border-gray-300 p-1">'_student'</td>
              <td class="border border-gray-300 p-1">1236</td>
              <td class="border border-gray-300 p-1 tracking-tighter">[-0.0460, 0.0031, 0.0545, ...]</td>
            </tr>
          </tbody>
        </table>
        <div class="mt-3 text-xs font-semibold">Vocabulary</div>
      </div>
      
    </div>
  </div>
</template>