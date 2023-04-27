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

<h1> Birds Classification Demo </h1> 

<label for="avatar">Upload a Bird Image:</label>
<input
	accept="image/png, image/jpeg"
	bind:files
	id="avatar"
	name="avatar"
	type="file"
/>

{#if imageUrl}
    <img src={imageUrl} alt={SUCCESS ? `A photo of an ${RN_pred} type bird.` : `An uploaded photo.`}/> 
{/if}

{#if SUCCESS}
    <h1> Common Name: {RN_pred}, Sci. Name: {SN_pred} </h1>
{:else if SUCCESS == false}
    <h1> Unfortunately, ML model no work on this. </h1>
{/if}


