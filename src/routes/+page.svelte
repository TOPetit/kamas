<script lang="ts">
    let regex: RegExp = /(\d+)\s+x\s+\[(.+?)\]\s+\(([\d\s]+)\s+kamas\)/;
    let chatDump: string = "";

    type resource = {
        name: string;
        count: number;
        totalPrice: number;
    };
    let resources: resource[] = [];
    let totalKamas: number = 0;

    function processLine(resource: resource) {
        if (resources.find((element) => element.name === resource.name)) {
            let index = resources.findIndex(
                (element) => element.name === resource.name,
            );
            resources[index].count += resource.count;
            resources[index].totalPrice += resource.totalPrice;
        } else {
            resources.push(resource);
        }
    }

    $: {
        if (chatDump) {
            resources = [];
            totalKamas = 0;
            let lines = chatDump.split("\n");
            lines.push("");
            let matches = lines
                .map((line) => line.match(regex))
                .filter(Boolean);
            matches.forEach((element) => {
                if (element) {
                    let [_, count, name, price] = element;
                    let resource: resource = {
                        name: name,
                        count: parseInt(count),
                        totalPrice: parseInt(price.replace(/\D/g, "")),
                    };
                    processLine(resource);
                }
            });
            totalKamas = resources.reduce(
                (acc, curr) => acc + curr.totalPrice,
                0,
            );
        } else {
            resources = [];
            totalKamas = 0;
        }
    }

    function adjustHeight(event: Event) {
        const textarea = event.target as HTMLTextAreaElement | null;
        if (textarea) {
            textarea.style.height = "auto";
            textarea.style.height = textarea.scrollHeight + "px";
        }
    }

    function prettyPrintNumber(num: number): string {
        return num.toLocaleString("fr-FR");
    }

    function handlePaste(event: ClipboardEvent) {
        const clipboardData = event.clipboardData;
        if (clipboardData) {
            const pastedData = clipboardData.getData("Text");
            chatDump += pastedData;
            window.scrollTo(0, 0); // Scroll to the top of the page

            // Scroll the textarea to the top
            const textarea = event.target as HTMLTextAreaElement;
            if (textarea) {
                textarea.scrollTop = 0;
            }
        }
    }
</script>

<div class="container">
    <div class="left-content">
        <ul>
            {#each resources as resource}
                <li>
                    {resource.count} x {resource.name}: {prettyPrintNumber(
                        resource.totalPrice,
                    )} kamas
                </li>
            {/each}
        </ul>
        {#if totalKamas > 0}
            <p>Total: {prettyPrintNumber(totalKamas)} kamas</p>
        {/if}
    </div>
    <textarea
        class="large-input"
        bind:value={chatDump}
        on:input={adjustHeight}
        on:paste={handlePaste}
        spellcheck="false"
        autocomplete="off"
    ></textarea>
</div>

<style>
    .container {
        display: flex;
        width: 100%;
    }
    .large-input {
        width: 50%;
        font-size: 1em;
        overflow: hidden;
        resize: none;
    }
    .left-content {
        width: 50%;
        padding: 10px;
    }
</style>
