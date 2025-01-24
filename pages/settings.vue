<template>
    <main id="settings" class="pl-8 pt-8 overflow-auto">
        <h1 class="text-4xl font-bold">Settings</h1>
        <span class="text-dim-3"> Change the behaviour of the application here. </span>

        <section id="theme-settings">
            <h2 class="mt-6 mb-2 text-2xl font-bold">Theme</h2>
            <div id="theme-selection" class="flex flex-row gap-4">
                <div
                    class="bg-elevation-1 bg-elevation-2-hover flex w-36 cursor-pointer flex-col items-center justify-center rounded-md p-2 text-xl font-semibold"
                    @click="setTheme('light')"
                >
                    <SunIcon class="h-8 w-8" :class="themeIconClass('light')" />
                    <label for="light-mode-icon" class="cursor-pointer">Light</label>
                </div>

                <div
                    class="bg-elevation-1 bg-elevation-2-hover flex w-36 cursor-pointer flex-col items-center justify-center rounded-md p-2 text-xl font-semibold"
                    @click="setTheme('dark')"
                >
                    <MoonIcon class="h-8 w-8" :class="themeIconClass('dark')" />
                    <label for="dark-mode-icon" class="cursor-pointer">Dark</label>
                </div>

                <div
                    class="bg-elevation-1 bg-elevation-2-hover flex w-36 cursor-pointer flex-col items-center justify-center rounded-md p-2 text-xl font-semibold"
                    @click="setTheme('catppuccin')"
                >
                    <IconCatppuccin class="h-8 w-8" :class="themeIconClass('catppuccin')" />
                    <label for="catppuccin-mode-icon" class="cursor-pointer">Catppuccin</label>
                </div>

                <div
                    class="bg-elevation-1 bg-elevation-2-hover flex w-36 cursor-pointer flex-col items-center justify-center rounded-md p-2 text-xl font-semibold"
                    @click="setTheme('custom')"
                >
                    <SwatchIcon class="h-8 w-8" :class="themeIconClass('custom')" />
                    <label for="custom-mode-icon" class="cursor-pointer">Custom</label>
                </div>
            </div>

            <div v-if="themeEditorDisplayed" class="mt-6 text-lg">
                <h3 class="mb-2 font-semibold">Select the colors for your custom theme:</h3>
                <CustomThemeEditor />
            </div>

            <button @click="$router.go(0)" class="mt-2 text-dim-3">
                If the colors do not update, please click <span class="underline">here</span>.
            </button>
        </section>

        <section id="miscellaneous-settings">
            <h2 class="mt-8 mb-2 text-2xl font-bold">Miscellaneous</h2>

            <div class="flex flex-col gap-4">
                <div class="flex w-[48rem] flex-row items-start justify-between">
                    <div>
                        <h3 class="text-lg">Export data to JSON</h3>
                        <span class="text-dim-2">
                            Backup all of your data (boards and themes) to a local JSON file.</span
                        >
                    </div>
                    <button
                        class="text-buttons bg-accent rounded-md px-4 py-2"
                        @click="exportJSON()"
                    >
                        Export
                    </button>
                </div>

                <div class="flex w-[48rem] flex-row items-start justify-between mb-8">
                    <div>
                        <h3 class="text-lg">Delete all data (themes and boards)</h3>
                        <span class="text-dim-2"
                            ><span class="text-red-500">Caution!</span> This will unreversibly
                            delete all of your data!</span
                        >
                    </div>
                    <button
                        class="text-buttons bg-accent rounded-md px-4 py-2"
                        @click="deleteAllData()"
                    >
                        Delete
                    </button>
                </div>
            </div>
        </section>
    </main>
</template>

<script setup lang="ts">
import { ask, message, save } from "@tauri-apps/api/dialog";
import { writeTextFile } from "@tauri-apps/api/fs";

import { useTauriStore } from "@/stores/tauriStore";
import { light, dark, catppuccin } from "@/utils/themes.js";
import emitter from "@/utils/emitter";

import { SwatchIcon, MoonIcon, SunIcon } from "@heroicons/vue/24/outline";

import { Ref } from "vue";
import { ThemeIdentifiers } from "~/types/kanban-types";

const store = useTauriStore().store;

const activeTheme: Ref<string | null> = ref("");
const themeEditorDisplayed = ref(false);

onMounted(async () => {
    activeTheme.value = await store.get("activeTheme");
    if (activeTheme.value === "custom") themeEditorDisplayed.value = true;
});

const setTheme = (themeName: ThemeIdentifiers) => {
    activeTheme.value = themeName;
    themeEditorDisplayed.value = false;

    const themes = { light, dark, catppuccin };

    if (themeName === "custom") {
        themeEditorDisplayed.value = true;
        return;
    }

    store.set("activeTheme", themeName);
    store.set("colors", themes[themeName]);
    emitter.emit("updateColors");
};

const themeIconClass = (theme: string) => {
    if (theme === activeTheme.value) return "text-accent";
    else return "";
};

const deleteAllData = async () => {
    const yes = await ask(
        "This action will irreversibly delete all of your boards, custom themes and revert all settings to default. Are you sure?",
        { title: "Kanri", type: "warning" }
    );

    if (yes) {
        store.delete("boards");
        store.delete("colors");
        store.set("activeTheme", "dark");

        await message("Successfully deleted data.", { title: "Kanri", type: "info" });
    } else return;
};

const exportJSON = async () => {
    const filePath = await save({
        title: "Select file to export data to",
        defaultPath: "./kanri_data_export.json",
        filters: [
            {
                name: "JSON File",
                extensions: ["json"],
            },
        ],
    });

    const savedBoards = await store.get("boards");
    const activeTheme = await store.get("activeTheme");
    const colors = await store.get("colors");

    const fileContents = JSON.stringify(
        {
            boards: savedBoards,
            activeTheme: activeTheme,
            colors: colors,
        },
        null,
        2
    );

    if (filePath == null) return;
    await writeTextFile(filePath, fileContents);

    // TODO: allow importing from JSON
};
</script>
