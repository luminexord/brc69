![BANNER_BRC69c](https://github.com/luminexord/brc69/assets/23469584/47753e6c-7654-432a-93d7-e79fee5f8128)

# BRC69

We propose a new standard for Ordinals Collections that utilize Recursive Inscription to optimize the costs of inscribing on Bitcoin using the Ordinals protocol. Additionally, this standard paves the way for more intriguing on-chain features, such as pre-reveal collection launching and on-chain reveals. This is accomplished by automatically and seamlessly rendering images on the Ordinals explorer, without the need for additional action.

---

## Context

As the Ordinals protocol gains traction, more users are inscribing data into Non-Fungible Ordinals collections on Bitcoin. This increased usage has led to a higher demand for Bitcoin block space, subsequently causing a rise in Bitcoin network fees. In order to continue encouraging creators to launch their innovative ideas on the Bitcoin blockchain, we must optimize the current standard for launching Image Ordinals Collections.

---

## Idea

We propose a new standard for launching Non-Fungible Ordinals Collection. This standard preserves all on-chain resources while achieving a 90%+ optimization of block space, which is dependent on the size of the initial collection and the network fees. The process involves four steps:

1. Inscribing the images of the traits on-chain
2. Inscribing the BRC69 collection deployment JSON
3. Inscribing the BRC69 collection compiler JavaScript
4. Inscribing the BRC69 asset with the mint operation

All these processes can be conducted without needing an external indexer, as long as the collection creators release the official list of inscriptions for their collections, as currently required. Moreover, the images will be automatically rendered on all front-end interfaces that have already implemented [Recursive Inscriptions](https://github.com/ordinals/ord/pull/2167), eliminating the need for additional steps.

---

## Operations

### Deploy BRC69

Once the images of the traits that comprise the collection are inscribed on-chain, we can inscribe the collection deploy JSON in the Deploy operation.

The Deploy operation is a JSON/Text inscription that contains general information about the collection and an array of the inscription IDs of the traits. The deploy inscription serves as the reference and the definitive source for the traits.

*Example of a collection deploy json:*

```javascript
{
   "p":"brc69",
   "op":"deploy",
   "collection":{
      "slug":"ordibots",
      "name":"OrdiBots",
      "description":"OrdiBots Testing Collection",
      "creator":"Martian",
      "supply":1000
   },
   "attributes":[
      "48f37de96ae78d693a0dd4022f0b09304a5389ebfc157c225399635b5cd1a84bi0",
      "b236b29525834dba35d61f4c719b8491047bc29f8eff219c7cc5062e6a1c340ai0",
      "669eb48d997b112729a6485f09a475d9869a5d547919e5ddd3f918f89360ed14i0",
      "e012b79a7b094920d5c120ea7c6fb44276346442721e6468390bedccdd07fc32i0",
      "6d2fa6c950a36d1e3be1df914a3458677b602a36c3a97d283ea7fa7123ced645i0",
      "d30730faf64535bb4b0744a4857eea5db7c9830fb0bb1551dd2fe69c9d5a1a87i0",
      "fb8366d635c530dd0a1e8e7b42ef7c04984b21489075913e1d480a11b811ba89i0",
      "a37ae051e2124e762f0a1d34a02fa10c05dd47eadd3520b08e94f84f61eb8093i0",
     // ... More traits inscription IDs
   ]
}
```

| Key        | Required | Description                                                  |
| ---------- | -------- | ------------------------------------------------------------ |
| p          | YES      | Protocol: Helps other systems identify and process brc69 events |
| op         | YES      | Operation: Type of event (Deploy, Compile, Mint)             |
| slug       | YES      | Slug: Identifier of the collection. Not enforced if no indexer implemented |
| name       | NO       | Name: Human readable name of the collection                  |
| supply     | NO       | Supply: Supply of the collection. Not enforced if no indexer implemented |
| attributes | YES      | Array of the inscription IDs of the traits that will generate the final assets |

### Compile BRC69

The Compile operation stores the logic to render the final assets in a JavaScript inscription. The Compile inscription is a recursive inscription that points back to the Deploy inscription to receive the traits' inscription IDs and ultimately render the asset. The logic of the Compile inscription can be customized for collections that require more specific rendering features.

Down belove we propose a versatile compile logic. 

``````javascript
/*
{
  "p": "brc69",
  "op": "compile",
  "s": "ordibots"
}
*/

// EDIT
const collectionJsonUrl = '/content/<deploy inscription id>';
const previewUrl = `/content/<preview inscription id>` // if preview available
const imageRendering = 'auto' // or pixelated
const renderSize = { width: 500, height: 500 }; // select image render size

async function loadImage (url) {
    return new Promise((resolve, reject) => {
        const image = document.createElement('img')
        image.src = url
        image.crossOrigin = 'anonymous'
        image.onload = () => {
            resolve(image)
        }
        image.onerror = () => {
            // Some display fallbacks for when the image fails to load
            if (!image.src.startsWith('https://')) {
                image.src = 'https://ordinals.com' + url
            } else if (image.src.startsWith('https://ordinals.com')) {
                image.src = 'https://ord-mirror.magiceden.dev' + url
            }
        }
    })
}

async function renderImage(imageEl, urls) {
    const canvas = document.createElement('canvas');
    canvas.width = renderSize.width;
    canvas.height = renderSize.height;

    const ctx = canvas.getContext("2d");
    ctx.imageSmoothingEnabled = false;

    const images = await Promise.all((urls).map(loadImage))
    images.forEach(_ => ctx.drawImage(_, 0, 0, canvas.width, canvas.height))
    imageEl.src = canvas.toDataURL("image/png")
}

async function getAllTraits(traitsUrl, retry = false) {
    try {
        const collectionMetadataRes = await fetch(traitsUrl)
        const collectionMetadata = await collectionMetadataRes.json()
        return collectionMetadata.attributes.map(_ => `/content/${_}`)
    } catch (e) {
        if (!retry) {
            const timestamp = Math.floor(Date.now() / (60000 * 10)) // 10 minutes
            const newTraitsUrl = `${traitsUrl}?timestamp=${timestamp}`
            return getAllTraits(newTraitsUrl, true)
        }
        throw e
    }
}

function createInitialImage () {
    // Manipulate the <body> tag
    document.body.style.margin = '0px';
    document.body.style.padding = '0px';

    // Create and set properties of the <img> tag
    const img = document.createElement('img');
    img.id = 'img';
    img.style.height = '100%';
    img.style.width = '100%';
    img.style.objectFit = 'contain';
    img.style.imageRendering = imageRendering;

    return img
}

async function createInscriptionHtml() {
    const imageEl = createInitialImage()

    try {
        // Get traits
        const allTraits = await getAllTraits(collectionJsonUrl)

        // Process traits
        const selectedTraitIndexes = document.querySelector('script[t]').getAttribute('t').split(',');
        const traits = selectedTraitIndexes.map(_ => allTraits[+_])

        // Render traits
        await renderImage(imageEl, traits);
    } catch (e) {
        console.error(e)

        // Render previewUrl image
        if (previewUrl) {
            imageEl.src = previewUrl
        }
    } finally {
        // Append the <img> tag to the <body>
        document.body.appendChild(imageEl);
    }
}

window.onload = function() {
    createInscriptionHtml();
}

``````

### Mint BRC69

The Mint operation uses an HTML type inscription that stores the index of the traits used to generate the final asset and points back to the Compile inscription, in a single line. This approach allows any front-end with Recursive Inscription to automatically render the image using on-chain inscribed data.

``````html
<script t="10,1,2,3" src="/content/<compile inscription id>" m='{"p":"brc69" "op":"mint" "s":"ordibots" "id":"0"}' ></script>
``````

| Key  | Required | Description                                                  |
| ---- | -------- | ------------------------------------------------------------ |
| t    | YES      | Traits: index of the traits used to generate the asset found in the "attributes" array of the deploy inscription |
| src  | YES      | Source: Recursive inscription pointer to the Compile inscription |
| m    | NO       | Metadata: metadata used to track BRC69 operations. Optional if you want your mints operation to be stealth |

---

## Impact

Implementing the BRC69 standard will enhance the efficiency of Bitcoin block space utilization. As the unique trait images are inscribed only once in the Deploy transaction, the assets are composed of an HTML file referencing these traits in just a single line of about 150 bytes. Any front-end with Recursive Inscription implementation can render the images without any additional steps, using the on-chain Deploy inscription.

---





