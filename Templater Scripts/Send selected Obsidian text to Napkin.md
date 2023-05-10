<%*
// Updated 2023-05-10

// Description: Sends selected text in Obsidian to Napkin

// Prerequisites
// 1. Templater Plugin installed in Obsidian https://obsidian.md/plugins?id=templater-obsidian
// 2. Active Napkin Account. Sign up here: https://www.napkin.one/?via=TfTHacker

// SETUP
// Add your Napkin Token and email address you registered with Napkin to the following parameters
const NAPKING_INFO = {
    token: "",
    email: ""
}

// The following text string will be inserted into Obsidian for ideas sent to Napkin.
// NAPKIN_URL will be replaced with the URL received by Napkin. 
// You can replace any text with what you want, just leave NAPKIN_URL
// If you don't want anything to be inserted, just have “” with nothing in the quotes.
const LINK_TO_NAPKIN_IDEA = " [*](NAPKIN_URL) #napkin_idea";

async function apiNapkinSendThought(newThought, source) {
    try {
        return await tp.obsidian.request({
            url: "https://app.napkin.one/api/createThought",
            method: "POST",
            contentType: "application/json",
            body: JSON.stringify({
                token: NAPKING_INFO.token,
                email: NAPKING_INFO.email,         
                thought: newThought,
                sourceUrl: source,
            }),
            throw: true
        })
    } catch(e) {
        return null; // Likely the token or email in not valid
    }
}

async function createNapkinAPIThoughtFromSelection(){
    const character_limit = 500 // Character limit for the message. 
    const notice_milliseconds = 5000 // number of milliseconds to show a notice

    if(app.workspace.activeEditor.currentMode.type!="source") {
        new Notice(`Napkin: document is not in source mode.`, notice_milliseconds);
        return;
    }
    
    let selection = app.workspace.activeEditor.getSelection();  // Get currently selected text in Editor
    if(selection.length===0) {
        new Notice(`Napkin: no text is selected.`, notice_milliseconds);
        return;
    }
    if(selection.length>NAPKING_INFO.character_limit) {
        new Notice(`Napkin: The selection is too long. Please limit the selection to ${NAPKING_INFO.character_limit} characters.`, notice_milliseconds);
        return;
    }

    const deeplink = "obsidian://open?"+
                        "vault=" + encodeURIComponent(app.vault.getName()) +
                        "&file=" + encodeURIComponent(tp.config.active_file.path);

    console.log("Sending ...")
    console.log("Idea:", selection)
    console.log("source:", deeplink)                        
    let response = await apiNapkinSendThought(selection, deeplink);
    console.log(response);
    if(response===null) {
        new Notice(`Napkin: The text was not sent to Napkin. Likely the token or email in not valid. Please check these values in the template script.`, notice_milliseconds)
    } else {
        // Success posting to the Napkin API. Make a note in the document
        if(LINK_TO_NAPKIN_IDEA==="") new Notice(`Napkin: The idea was saved to Napkin.`, notice_milliseconds);
        selection = selection + LINK_TO_NAPKIN_IDEA.replace("NAPKIN_URL", JSON.parse(response).url);
    }

    app.workspace.activeEditor.editor.replaceSelection(selection);
}

await createNapkinAPIThoughtFromSelection()

%>