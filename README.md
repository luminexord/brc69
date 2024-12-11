# BRC69

We propose a standard for Ordinals Collections that utilizes Recursive Inscription to optimize the costs of inscribing on Bitcoin using the Ordinals protocol. Additionally, this standard paves the way for more intriguing on-chain features, such as pre-reveal collection launching and on-chain reveals. This is accomplished by automatically and seamlessly rendering images on the Ordinals explorer, without the need for additional action.

---

## Context

As the Ordinals protocol gains traction, more users are inscribing data into Non-Fungible Ordinals collections on Bitcoin. This increased usage has led to a higher demand for Bitcoin block space, subsequently causing a rise in Bitcoin network fees. In order to continue encouraging creators to launch their innovative ideas on the Bitcoin blockchain, we must optimize the current standard for launching Image Ordinals Collections.

---

## Idea

We propose a new standard for launching Non-Fungible Ordinals Collection. This standard preserves all on-chain resources while achieving a 90%+ optimization of block space, which is dependent on the size of the initial collection and the network fees. The process involves three steps:

1. Inscribing the images of the traits on-chain
2. Inscribing the BRC69 collection deployment JSON
3. Inscribing the BRC69 assets with the mint operation

All these processes can be conducted without needing an external indexer, as long as the collection creators release the official list of inscriptions for their collections, as currently required. Moreover, the images will be automatically rendered on all front-end interfaces that have already implemented [Recursive Inscriptions](https://luminex.gitbook.io/luminex/ordinals/recursive-inscriptions), eliminating the need for additional steps.

---

## Operations

### Deploy BRC69

Once the images of the traits that comprise the collection are inscribed on-chain, we can inscribe the collection deploy JSON in the Deploy operation.

The Deploy operation is a JSON/Text inscription that contains general information about the collection, compiler settings, and an array of the traits with their inscription IDs. The deploy inscription serves as the reference and the definitive source for the traits.

*Example of a collection deploy json:*

```javascript
{
   "p": "brc69",
   "op": "deploy",
   "collection": {
      "slug": "collection-name"
   },
   "compilerInfo": {
      "preview": "",  // optional preview inscription id
      "renderSize": {
         "width": 750, // width dimension rendered
         "height": 750 // heigh dimension rendered
      },
      "imageRendering": "auto"  // or "pixelated"
   },
   "attributes": [
      {
         "traitType": "Background",
         "order": 0,
         "traits": [
            ["Blue", 50, "/content/trait_inscription_id_1"],
            ["Red", 50, "/content/trait_inscription_id_2"]
         ]
      },
      // More trait categories...
   ]
}
```

| Key         | Required | Description                                                  |
| ----------- | -------- | ------------------------------------------------------------ |
| p           | NO      | Protocol: Helps other systems identify and process brc69 events |
| op          | NO      | Operation: Type of event (Deploy)                            |
| slug        | NO      | Slug: Identifier of the collection                           |
| compilerInfo| YES      | Settings for the standard compiler                           |
| attributes  | YES      | Array of trait categories with their inscription IDs and rarities |

### Mint BRC69

The Mint operation uses an HTML type inscription that stores the index of the traits used to generate the final asset and points back to the standard compiler inscription. This approach allows any front-end with Recursive Inscription to automatically render the image using on-chain inscribed data.

```html
<script t="0,3,10,5,3,14,10,25" src="/content/<compiler_inscription_id>" d="/content/<deployer_inscription_id>"></script>
```

| Key  | Required | Description                                                  |
| ---- | -------- | ------------------------------------------------------------ |
| t    | YES      | Traits: index of the traits used to generate the asset        |
| src  | YES      | Source: Points to the standard compiler inscription           |
| d    | YES      | Deploy: Points to your collection's deploy inscription        |

Current Standard Compiler Inscription ID: `4539667b4a45072aef54b40e4e6d8e288636d68e58f46414f611f01b40ec2039i0`
Last Updated: October 31, 2024

---

## Impact

Implementing the BRC69 standard will enhance the efficiency of Bitcoin block space utilization. As the unique trait images are inscribed only once in the Deploy transaction, the assets are composed of an HTML file referencing these traits in just a single line of about 150 bytes. Any front-end with Recursive Inscription implementation can render the images without any additional steps, using the on-chain Deploy inscription.

## Easy Creation

For creators without coding experience, collections can now be created using the Luminex Creator page at https://luminex.io/ordinals/launch. This tool provides a user-friendly interface for creating BRC69 collections without requiring any technical knowledge.

---