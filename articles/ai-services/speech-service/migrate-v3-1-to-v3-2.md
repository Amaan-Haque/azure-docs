---
title: Migrate from v3.1 to v3.2 REST API - Speech service
titleSuffix: Azure AI services
description: This document helps developers migrate code from v3.1 to v3.2 of the Speech to text REST API.
services: cognitive-services
author: eric-urban
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: how-to
ms.date: 09/15/2023
ms.author: eur
ms.devlang: csharp
ms.custom: devx-track-csharp
---

# Migrate code from v3.1 to v3.2 of the REST API

The Speech to text REST API is used for [Batch transcription](batch-transcription.md) and [Custom Speech](custom-speech-overview.md). Changes from version 3.1 to 3.2 are described in the sections below.

> [!IMPORTANT]
> Speech to text REST API v3.2 is available in preview. 
> [Speech to text REST API](rest-speech-to-text.md) v3.1 is generally available. 
> Speech to text REST API v3.0 will be retired on April 1st, 2026. For more information, see the Speech to text REST API [v3.0 to v3.1](migrate-v3-0-to-v3-1.md) and [v3.1 to v3.2](migrate-v3-1-to-v3-2.md) migration guides.


## Base path

You must update the base path in your code from `/speechtotext/v3.1` to `/speechtotext/v3.2-preview.1`. For example, to get base models in the `eastus` region, use `https://eastus.api.cognitive.microsoft.com/speechtotext/v3.2-preview.1/models/base` instead of `https://eastus.api.cognitive.microsoft.com/speechtotext/v3.1/models/base`.

For more information, see [Operation IDs](#operation-ids) later in this guide.

## Batch transcription

> [!IMPORTANT]
> New pricing is in effect for batch transcription via Speech to text REST API v3.2. For more information, see the [pricing guide](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services).

### Backwards compatibility limitations

Don't use Speech to text REST API v3.0 or v3.1 to retrieve a transcription created via Speech to text REST API v3.2. You'll see an error message such as the following: "The API version can't be used to access this transcription. Please use API version v3.2 or higher."

### Language identification mode

The `LanguageIdentificationMode` is added to `LanguageIdentificationProperties` as sibling of `candidateLocales` and `speechModelMapping`. The modes available for language identification are `Continuous` or `Single`. Continuous language identification is the default. For more information, see [Language identification](./language-identification.md#at-start-and-continuous-language-identification).

### Whisper models

Azure AI Speech now supports OpenAI's Whisper model via Speech to text REST API v3.2. To learn more, check out the [Create a batch transcription](./batch-transcription-create.md#using-whisper-models) guide. 

> [!NOTE]
> Azure OpenAI Service also supports OpenAI's Whisper model for speech to text with a synchronous REST API. To learn more, check out the [quickstart](../openai/whisper-quickstart.md). Check out [What is the Whisper model?](./whisper-overview.md) to learn more about when to use Azure AI Speech vs. Azure OpenAI Service. 

## Custom Speech

> [!IMPORTANT]
> You'll be charged for custom speech model training if the base model was created on October 1, 2023 and later. You are not charged for training if the base model was created prior to October 2023. For more information, see [Azure AI Speech pricing](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).
> 
> To programmatically determine whether a model was created before or after October 1, 2023, use the `chargedForAdaptation` property that's [new in version 3.2](#charge-for-adaptation).

### Charge for adaptation

The `chargeForAdaptation` property is added to `BaseModelProperties`. This is within the `BaseModel` definition.

> [!IMPORTANT]
> You'll be charged for custom speech model training if the base model was created on October 1, 2023 and later. You are not charged for training if the base model was created prior to October 2023. For more information, see [Azure AI Speech pricing](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

If the value of `chargeForAdaptation` is `true`, you're charged for training the model. If the value is `false`, you're charged for training the model. Use the `chargeForAdaptation` property instead of the created date to programmatically determine whether you're charged for training a model. 

```json
"BaseModelProperties": {
    "title": "BaseModelProperties",
    "type": "object",
    "properties": {
        "deprecationDates": {
            "$ref": "#/definitions/BaseModelDeprecationDates"
        },
        "features": {
            "$ref": "#/definitions/BaseModelFeatures"
        },
        "chargeForAdaptation": {
            "description": "A value indicating whether model adaptation is charged.",
            "type": "boolean",
            "readOnly": true
        }
    }
},
```

### Text normalization

The `textNormalizationKind` property is added to `DatasetProperties`.
 
Entity definition for TextNormalizationKind: The kind of text normalization.
- Default: Default text normalization (for example, 'two to three' replaces '2 to 3' in en-US).
- None: No text normalization is applied to the input text. This value is an override option that should only be used when text is normalized before the upload.

### Evaluation properties

Added token count and token error properties to the `EvaluationProperties` properties:
- `correctTokenCount1`: The number of correctly recognized tokens by model1.
- `tokenCount1`: The number of processed tokens by model1.
- `tokenDeletionCount1`: The number of recognized tokens by model1 that are deletions.
- `tokenErrorRate1`: The token error rate of recognition with model1. 
- `tokenInsertionCount1`: The number of recognized tokens by model1 that are insertions.
- `tokenSubstitutionCount1`: The number of recognized words by model1 that are substitutions.
- `correctTokenCount2`: The number of correctly recognized tokens by model2.
- `tokenCount2`: The number of processed tokens by model2.
- `tokenDeletionCount2`: The number of recognized tokens by model2 that are deletions.
- `tokenErrorRate2`: The token error rate of recognition with model2. 
- `tokenInsertionCount2`: The number of recognized tokens by model2 that are insertions.
- `tokenSubstitutionCount2`: The number of recognized words by model2 that are substitutions.

### Model copy

Added the new `"/operations/models/copy/{id}"` operation. Used for copy models scenario.

Added the new `"/models/{id}:copy"` operation. Schema in the new copy operation: `"$ref": "#/definitions/ModelCopyAuthorization"` Deprecated the `"/models/{id}:copyto"` operation. Schema in the deprecated copy operation: `"$ref": "#/definitions/ModelCopy"`

Added the new `"/models:authorizecopy"` operation returns `"$ref": "#/definitions/ModelCopyAuthorization"`. This returned entity can be used in the new `"/models/{id}:copy"` operation.

New entity definitions related to model copy authorization:
- `ModelCopyAuthorization`
- `ModelCopyAuthorizationDefinition`: The Azure Resource ID of the source speech resource.

```json
"ModelCopyAuthorization": {
    "title": "ModelCopyAuthorization",
    "required": [
        "expirationDateTime",
        "id",
        "sourceResourceId",
        "targetResourceEndpoint",
        "targetResourceId",
        "targetResourceRegion"
    ],
    "type": "object",
    "properties": {
        "targetResourceRegion": {
            "description": "The region (aka location) of the target speech resource (e.g., westus2).",
            "minLength": 1,
            "type": "string"
        },
        "targetResourceId": {
            "description": "The Azure Resource ID of the target speech resource.",
            "minLength": 1,
            "type": "string"
        },
        "targetResourceEndpoint": {
            "description": "The endpoint (base url) of the target resource (with custom domain name when it is used).",
            "minLength": 1,
            "type": "string"
        },
        "sourceResourceId": {
            "description": "The Azure Resource ID of the source speech resource.",
            "minLength": 1,
            "type": "string"
        },
        "expirationDateTime": {
            "format": "date-time",
            "description": "The expiration date of this copy authorization.",
            "type": "string"
        },
        "id": {
            "description": "The ID of this copy authorization.",
            "minLength": 1,
            "type": "string"
        }
    }
},
```

```json
"ModelCopyAuthorizationDefinition": {
    "title": "ModelCopyAuthorizationDefinition",
    "required": [
        "sourceResourceId"
    ],
    "type": "object",
    "properties": {
        "sourceResourceId": {
            "description": "The Azure Resource ID of the source speech resource.",
            "minLength": 1,
            "type": "string"
        }
    }
},
```

### CustomModelLinks copy properties

New `copy` property
copyTo URI: The location to the obsolete model copy action. See operation \"Models_CopyTo\" for more details.
copy URI: The location to the model copy action. See operation \"Models_Copy\" for more details.

```json
"CustomModelLinks": {
    "title": "CustomModelLinks",
    "type": "object",
    "properties": {
      "copyTo": {
        "format": "uri",
        "description": "The location to the obsolete model copy action. See operation \"Models_CopyTo\" for more details.",
        "type": "string",
        "readOnly": true
      },
      "copy": {
        "format": "uri",
        "description": "The location to the model copy action. See operation \"Models_Copy\" for more details.",
        "type": "string",
        "readOnly": true
      },
      "files": {
        "format": "uri",
        "description": "The location to get all files of this entity. See operation \"Models_ListFiles\" for more details.",
        "type": "string",
        "readOnly": true
      },
      "manifest": {
        "format": "uri",
        "description": "The location to get a manifest for this model to be used in the on-prem container. See operation \"Models_GetCustomModelManifest\" for more details.",
        "type": "string",
        "readOnly": true
      }
    },
    "readOnly": true
},
```

## Operation IDs

You must update the base path in your code from `/speechtotext/v3.1` to `/speechtotext/v3.2-preview.1`. For example, to get base models in the `eastus` region, use `https://eastus.api.cognitive.microsoft.com/speechtotext/v3.2-preview.1/models/base` instead of `https://eastus.api.cognitive.microsoft.com/speechtotext/v3.1/models/base`.


## Next steps

* [Speech to text REST API](rest-speech-to-text.md)
* [Speech to text REST API v3.2 (preview) specification](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/cognitiveservices/data-plane/Speech/SpeechToText/preview/v3.2-preview.1)
* [Speech to text REST API v3.1 reference](https://eastus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-1)
* [Speech to text REST API v3.0 reference](https://eastus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)


