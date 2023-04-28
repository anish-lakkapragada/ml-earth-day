<script lang="ts"> 
    import { SupabaseClient, createClient } from '@supabase/supabase-js'; 
    const client : SupabaseClient = createClient(import.meta.env.VITE_SECRET_URL, import.meta.env.VITE_SECRET_PAK)

    // just some basic declarations
    let files: FileList | null; 
    let imageUrl : string | null;
    let SUCCESS : boolean | null;  
    let SN_pred : string | null; 
    let RN_pred : string | null; 

    $: if (files) {
        const file : File = files[0]; 
        (async () => {
            const { data, error } = await client.storage.from('birds-caa-event').upload(`public/birds/${Date.now()}`, file, { cacheControl: '5', upsert: true });
            console.log(data);
            console.log(`error is: ${error}`);
            if (error == undefined) {
                const {data : urlData} = client.storage.from("birds-caa-event").getPublicUrl(data.path);
                imageUrl = urlData.publicUrl;
                const predictionData = await predictImageType(imageUrl);
                console.log(predictionData);
                SUCCESS = predictionData.successClassification; 
                SN_pred = predictionData.classificationResp.SN; 
                RN_pred = predictionData.classificationResp.RN; 
            }
        })(); 
    }

    // extras 
    const ENDPOINT : string = "https://birds.lakkapragada.com";
    async function predictImageType(srcURL: string) {
        const data = await fetch(`${ENDPOINT}/predict?imageURL=${srcURL}`);
        const classificationResp = await data.json();
        let successClassification = true;
        console.log(classificationResp);
        if (
        classificationResp.error ||
        classificationResp?.SN.includes("background")
        ) {
            successClassification = false;
            console.log(classificationResp)
        }
        return {successClassification: successClassification, classificationResp: classificationResp};
    }
</script> 

<div class="main">

    <h1> Birds Classification Demo </h1> 

    <div class="label"> 
        <h2>Upload a Bird Image:</h2>
        <input
            accept="image/png, image/jpeg"
            bind:files
            type="file"
        />
    </div>

    {#if imageUrl}
        <img src={imageUrl} alt={SUCCESS ? `A photo of an ${RN_pred} type bird.` : `An uploaded photo.`}/> 
    {/if}

    <div class="classification-text"> 
        {#if SUCCESS}
            <h1> Common Name: {RN_pred} </h1> 
            <h1> Sci. Name: {SN_pred} </h1>
        {:else if SUCCESS == false}
            <h1> Unfortunately, ML model no work on this. </h1>
        {/if}
    </div> 

</div> 

<style> 

body {
    background-color: lavender;
}

img {
    height: 20em; 
    margin-top: 2em;
}

.label {
    margin-top: 5em;
}

.main {
    text-align: center; 
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    background-color: lavender;
    height: 2000px;
    margin: 0;
    top: 0;
}

.classification-text {
    margin-top: 2em;
    width: 100%; 
    justify-content: center;
    align-items: center;
    display: flex; 
    flex-direction: column;
    gap: 1em;
}

h1 {
    top: 0;
    margin: 0;
    padding: 0;
}

</style> 


