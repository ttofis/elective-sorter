<script>
    import { onMount } from "svelte";
    import { Stepper, Step, ProgressRadial } from "@skeletonlabs/skeleton";
    import Icon from '@iconify/svelte';
    import { flip } from "svelte/animate";

    let locked = true;
    let completed = false;

    let battle = 0;
    let complBattles = 0;
    let left = 0;
    let right = 0;
    let totalBattles = 0;

    let options = new Array(
        "Αγγειοχειρουργική",
        "Αιματολογία",
        "Αναισθησιολογία",
        "Γαστρεντερολογία",
        "Γενική Ιατρική",
        "Ενδοκρινολογία",
        "Εντατική Θεραπεία",
        "Ιατρική Γενετική",
        "Νευροχειρουργική",
        "Νεφρολογία",
        "Ογκολογία",
        "Ρευματολογία",
        "Χειρουργική Θώρακα-Καρδιάς",
        "Ψυχιατρική Παιδιού και Εφήβου (Παιδοψυχιατρική)",
        "Ακτινοθεραπευτική Ογκολογία",
        "Λοιμώξεις",
        "Παιδιατρική-Εξειδικεύσεις/Γνωστικά Αντικείμενα Παιδιατρική",
        "Πυρηνική Ιατρική",
        "Χειρουργική Παίδων",
        "Δερματολογία",
        "Καρδιολογία",
        "Νευρολογία",
        "Ορθοπαιδική-Τραυματολογία",
        "Ουρολογία",
        "Οφθαλμολογία",
        "Πνευμονολογία",
        "ΩΡΛ"
    )

    // Setup array
    let scoring = [];
    let battles = [];
    let scores = [];

    let progress;

    function isItemInArray(array, item) {
        for (var i = 0; i < array.length; i++) {
            if (array[i][0] == item[0] && array[i][1] == item[1]) {
                return i
            }
        }
        return -1;
    }

    function init() {
        for (let i = 0; i < options.length; i++) {
            scoring[i] = []
            scores[i] = [0, options[i]];
            for (let j = 0; j < options.length; j++) {
                scoring[i][j] = -2;
                if (i == j) scoring[i][j] = 0;
            }
        }
        for (let i = 1; i < options.length; i++) {
            for (let j = 0; j < options.length-i; j++) {
                battles.push([j,j+i]);
            }
        }

        totalBattles = battles.length;
        progress = Math.floor((battle/totalBattles)*100);
        decide();
    }

    function removeBattle(a) {
        battles.splice(a, 1);
        complBattles++;
    }

    function decide() {
        left = battles[0][0];
        right = battles[0][1];
    }

    function act(action) {
        switch (action) {
            case -1:
            case 0:
            case 1:
                battle++;
                complBattles++;
                battles.shift();
                process(left, right, action);
                break;
            case 2:
                battles.push(battles.shift());
                break;
        }
        if (battles.length != 0)  {
            decide()
        }else{
            locked = false;
            calculate();
        }
        progress = Math.floor((complBattles/totalBattles)*100);
    }

    function sortScores(a, b) {
        if (a[0] === b[0]) {
            return 0;
        }
        else {
            return (a[0] > b[0]) ? -1 : 1;
        }
    }

    function calculate() {
        for (let i = 0; i < options.length; i++) {
            for (let j = 0; j < options.length; j++) {
                scores[i][0] += scoring[i][j];
            }
        }
        scores.sort(sortScores);
        let x = scores[0][0];
        let rem = 1;
        scores[0][0] = 1;

        for (let i = 1; i < options.length; i++) {
            scores[i][0] = i+1;
            if (scores[i][0] == x) {
                scores[i][0] = rem;
            }else{
                x = scores[i][0];
                scores[i][0] = i+1;
                rem = scores[i][0];
            }
        }
    }

    // Needs rewriting and checking
    function process(x,y,act) {
        if (act != 0) {
            scoring[x][y] = act;
            scoring[y][x] = -act;
        }else{
            scoring[x][y] = 0;
            scoring[y][x] = 0;
        }

        
        if (act == 1) {
            let tmp = x;
            x = y;
            y = tmp;
        }
        if (act != 0) {
            for (let i = 0; i < options.length; i++) {
                if (scoring[x][i] == 1 || scoring[x][i] == 0) {
                    let find = isItemInArray(battles, [Math.min(y,i), Math.max(y,i)]);
                    if (find != -1) {
                        removeBattle(find);
                        process(y,i,1);
                    }
                }
                if (scoring[y][i] == -1  || scoring[y][i] == 0) {
                    let find = isItemInArray(battles, [Math.min(x,i), Math.max(x,i)]);
                    if (find != -1) {
                        removeBattle(find);
                        process(x,i,-1);
                    }
                }
            }
        }else{
            for (let i = 0; i < options.length; i++) {
                if (scoring[x][i] != 0 && scoring[x][i] != -2) {
                    let find = isItemInArray(battles, [Math.min(y,i), Math.max(y,i)]);
                    if (find != -1) {
                        removeBattle(find);
                        process(y,i,scoring[x][i]);
                    }
                }
                if (scoring[y][i] != 0  && scoring[y][i] != -2) {
                    let find = isItemInArray(battles, [Math.min(x,i), Math.max(x,i)]);
                    if (find != -1) {
                        removeBattle(find);
                        process(x,i,scoring[y][i]);
                    }
                }
            }
        }
    }
    onMount(() => {
        init();
    })

    function complete() {
        completed = true;
    }

    function move(x, where) {
        if (x == 0 && where == 1) return;
        if (x == options.length-1 && where == -1) return;
        let tmp = scores[x];
        scores[x] = scores[x-where];
        scores[x-where] = tmp;
    }
</script>

<div class="container mx-auto p-8 space-y-8 print:p-0 print:border-0">
    <section class="card p-4 variant-glass print:bg-white print:border-0">
        {#if !completed}
       <Stepper stepTerm="Βήμα" buttonBackLabel="Πίσω" buttonNextLabel="Επόμενο" buttonCompleteLabel="Τέλος" on:complete={() => complete()}>
            <Step>
                <svelte:fragment slot="header">Οδηγίες χρήσης</svelte:fragment>
                <p>Επίλεξε την προτίμησή σου μεταξύ δύο αντικειμένων και θα πάρεις πίσω την λίστα με την κατάταξη των προτιμήσεών σου.</p>
            </Step>
            <Step locked={locked}>
                <svelte:fragment slot="header">Επιλογή προτίμησης</svelte:fragment>
                <div class="container mx-auto px-4 grid grid-cols-2 gap-3">
                    <section class="col-span-2 card p-3 variant-glass grid grid-cols-3 sm:grid-cols-8 gap-2">
                        <div class="col-span-2 sm:col-span-7 flex items-center justify-center">
                            <p class="unstyled text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-medium">Μάχη #{battle}</p>
                        </div>
                        <ProgressRadial meter="stroke-primary-400 dark:stroke-primary-500" font="120" stroke="80" value={progress}>{progress}%</ProgressRadial>
                    </section>
                    <button disabled={!locked} on:click={() => act(1)} class="col-span-2 sm:col-span-1 card card-hover p-3 variant-glass hover:variant-glass-primary transition-colors">
                        <p class="unstyled text-base md:text-2xl font-bold">{#if locked}{options[left]}{:else}Τέλος{/if}</p>
                    </button>
                    <button disabled={!locked} on:click={() => act(-1)} class="col-span-2 sm:col-span-1 card card-hover p-3 variant-glass hover:variant-glass-primary transition-colors">
                        <p class="unstyled text-base md:text-2xl font-bold">{#if locked}{options[right]}{:else}Τέλος{/if}</p>
                    </button>
                    <button disabled={!locked} on:click={() => act(0)} class="col-span-2 sm:col-span-1 btn variant-filled-primary text-sm sm:text-base md:text-2xl">Μου αρέσουν και τα δύο</button>
                    <button disabled={!locked} on:click={() => act(2)} class="col-span-2 sm:col-span-1 btn variant-filled-error text-sm sm:text-base md:text-2xl">Δεν έχω άποψη</button>
                </div>
            </Step>
            <Step>
                <svelte:fragment slot="header">Αποτελέσματα</svelte:fragment>
                <div class="container mx-auto md:px-4">
                    <div class="relative overflow-x-auto shadow-md sm:rounded-lg">
                        <table class="w-full text-sm md:text-m lg:text-lg text-left">
                            <caption class="p-5 text-lg font-semibold text-left bg-surface-200 dark:bg-surface-600">
                                <p class="mt-1 text-sm font-normal">Τα ακόλουθα αποτελέσματα έχουν δημιουργηθεί βάση των απαντήσεων που έχεις δώσει. Κάνε όποιες αλλαγές επιθυμείς και συνέχισε στο επόμενο βήμα. Κάνε τυχόν διορθώσεις (πχ σε εκείνα που έχουν την ίδια εισήγηση κατάταξης) και πάτα Τέλος.</p>
                            </caption>
                            <thead class="bg-surface-300 dark:bg-surface-900">
                                <tr>
                                    <th scope="col" class="px-4 py-3">
                                        A/A
                                    </th>
                                    <th scope="col" class="px-4 py-3">
                                        Μάθημα
                                    </th>
                                    <th scope="col" class="px-4 py-3">
                                        <span class="sr-only">Edit</span>
                                    </th>
                                </tr>
                            </thead>
                            <tbody>
                                {#each scores as s, i (s)}
                                <tr animate:flip class="bg-surface-50 border-b border-surface-300 dark:bg-surface-700 dark:border-surface-900">
                                    <th scope="row" class="px-2 md:px-4 py-3 font-medium whitespace-nowrap">
                                        {i+1} ({s[0]})
                                    </th>
                                    <td class="px-2 md:px-4 py-3">
                                        {s[1]}
                                    </td>
                                    <td class="px-2 md:px-4 py-3 text-right">
                                        <button on:click={() => move(i,1)}><Icon inline={true} width="25" icon="material-symbols:keyboard-arrow-up" /></button>
                                        <button on:click={() => move(i,-1)}><Icon inline={true} width="25" icon="material-symbols:keyboard-arrow-down" /></button>
                                    </td>
                                </tr>
                                {/each}
                            </tbody>
                        </table>
                    </div>
                </div>
            </Step>
        </Stepper>
        {:else}
        <div class="print:hidden">
            <h1 class="pb-2">Τελικά αποτελέσματα</h1>
            <p  class="pb-2">Πάτησε <button on:click={() => window.print()} class="btn btn-sm variant-filled-primary">Εκτύπωση</button> για να εκτυπώσεις ή να αποθηκεύσεις σε μορφή pdf τα αποτελέσματα. Αν θες να το κάνεις ξανά από την αρχή, κάνε refresh.</p>
        </div>
        <div class="container mx-auto px-4 mt-2">
            <div class="relative overflow-x-auto shadow-md print:shadow-none sm:rounded-lg">
                <table class="w-full text-lg leading-5 print:leading-[0.3rem] text-left">
                    <thead class="bg-surface-300 dark:bg-surface-900 print:border-b">
                        <tr>
                            <th scope="col" class="px-6 py-3">
                                Κατάταξη
                            </th>
                            <th scope="col" class="px-6 py-3">
                                Μάθημα
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each scores as s, i (s)}
                        <tr class="bg-surface-50 border-b border-surface-300 dark:bg-surface-700 dark:border-surface-900">
                            <th scope="row" class="px-6 py-3 font-medium whitespace-nowrap">
                                {i+1}
                            </th>
                            <td class="px-6 py-3">
                                {s[1]}
                            </td>
                        </tr>
                        {/each}
                    </tbody>
                </table>
            </div>
        </div>
        {/if}
    </section>
</div>