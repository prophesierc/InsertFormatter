<template>
  <div class="h-screen w-screen overflow-auto">
    <div class="m-2 gap-2 flex justify-center">
      <Basebutton @click="addRow"> + Add Row </Basebutton>
      <Basebutton @click="addColumn"> + Add Column </Basebutton>
      <Basebutton @click="clear"> Clear </Basebutton>
      <Basebutton @click="reset"> Reset </Basebutton>
    </div>

    <div class="overflow-auto mx-4">
      <table class="border-collapse border border-slate-700 text-sm">
        <tbody>
          <tr v-for="(row, rowIndex) in rows" :key="rowIndex">
            <td
              v-for="columnIndex in row.length"
              :key="columnIndex"
              class="border border-slate-700 p-0"
            >
              <input
                v-model="row[columnIndex - 1]"
                :data-column="columnIndex - 1"
                class="h-9 w-[4.85vw] max-w-full bg-slate-900 px-2 text-slate-200 outline-none focus:bg-slate-800"
                @paste="paste($event, rowIndex, columnIndex - 1)"
              />
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="flex justify-center overflow-auto text-slate-200">
      <pre
        class="mt-10 h-[25vh] w-[80vw] resize both overflow-auto bg-slate-900 p-4 max-w-screen mx-4 mb-10"
        v-html="highlightedOutput"
        :key="textAreaSize"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import Basebutton from "@/components/base/Basebutton.vue";
import { ref, computed } from "vue";

const defaultRowCount = 10;
const defaultColumnCount = 20;
const textAreaSize = ref(0);

const rows = ref(
  Array.from({ length: defaultRowCount }, () =>
    Array.from({ length: defaultColumnCount }, () => ""),
  ),
);

const addRow = () => {
  rows.value.push(Array.from({ length: rows.value[0].length }, () => ""));
};

const addColumn = () => {
  rows.value.forEach((row) => row.push(""));
};

const clear = () => {
  rows.value.forEach((row) => {
    row.fill("");
  });
};

const reset = () => {
  clear();
  rows.value.length = defaultRowCount;
  rows.value.forEach((row) => {
    row.length = defaultColumnCount;
    row.fill("");
  });
  textAreaSize.value++;
};

const highlightedOutput = computed(() => {
  return output.value.replace(
    /VALUES\n([\s\S]*)/,
    '<span class="text-yellow-200">VALUES</span>\n<span class="text-green-400">$1</span>',
  );
});

const formatValue = (value: string) => {
  value = value.trim();
  if (value.startsWith("'") && value.endsWith("'")) {
    return value;
  }
  if (value.toUpperCase() === "NULL") {
    return "NULL";
  }
  if (/^-?\d+(\.\d+)?$/.test(value)) {
    return value;
  }
  return `'${value.replace(/'/g, "''")}'`;
};

const output = computed(() => {
  const filledRows = rows.value.filter((row) =>
    row.some((value) => value.trim()),
  );

  const lastColumn = Math.max(
    ...filledRows.map((row) =>
      row.reduce((last, value, index) => (value.trim() ? index : last), -1),
    ),
  );

  if (lastColumn < 0) return "";

  const values = filledRows.map(
    (row) =>
      `(${row
        .slice(0, lastColumn + 1)
        .map(formatValue)
        .join(", ")})`,
  );
  return `INSERT INTO table_name\nVALUES\n${values.join(",\n")};`;
});

const paste = (event: ClipboardEvent, row: number, column: number) => {
  event.preventDefault();
  const text = event.clipboardData?.getData("text/plain");
  if (!text) return;

  text
    .trimEnd()
    .split(/\r?\n/)
    .forEach((line, r) => {
      const targetRow = row + r;
      const values = line.split("\t");

      while (!rows.value[targetRow]) addRow();

      rows.value[targetRow].length = Math.max(
        rows.value[targetRow].length,
        column + values.length,
      );

      values.forEach((value, c) => {
        rows.value[targetRow][column + c] = value;
      });
    });
};
</script>
