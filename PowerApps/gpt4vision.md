# Power Apps accessing GPT 4 Vision

## Requirements

- Power platform environment
- Power automate or flow
- Azure Open AI gpt 4 vision in one of the available regions

## Steps to create power apps and flow

### Step 1 - Create the power automate flow

- Entire flow

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-1.jpg 'Mistralai')

- First lets get 2 parameters one for prompt and one for image data
- Initialize 2 variables
- ask from power apps to fill those 2 variables

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-2.jpg 'Mistralai')

- Now lets call the HTTP action to call the open ai gpt 4 vision
- Pass the prompt and image data
- Make sure JSON format is as below

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-3.jpg 'Mistralai')

- here is the Body

```
{
  "messages": [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": @{variables('prompt')}
        },
        {
          "type": "image_url",
          "image_url": {
            "url": @{concat('data:image/jpeg;base64,', variables('imagecontent'))}
          }
        }
      ]
    }
  ],
  "temperature": 0.7,
  "top_p": 0.95,
  "max_tokens": 4096
}
```

- Provide the deployment name and API key
- Set the content headers
- for URL use the below

```
https://{aoairesourcename}.openai.azure.com/openai/deployments/{deploymentname}/chat/completions?api-version=2023-07-01-preview
```

- provide the api key
- Also i appened the image data with base64 encoding to image content variable
- Now configure a output variable to hold the response from open ai

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-4.jpg 'Mistralai')

- Next is Parse the output from HTTP request using ParseJSON

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-5.jpg 'Mistralai')

- here is the schema used

```
{
    "type": "object",
    "properties": {
        "id": {
            "type": "string"
        },
        "object": {
            "type": "string"
        },
        "created": {
            "type": "integer"
        },
        "model": {
            "type": "string"
        },
        "prompt_filter_results": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "prompt_index": {
                        "type": "integer"
                    },
                    "content_filter_results": {
                        "type": "object",
                        "properties": {
                            "hate": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            },
                            "self_harm": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            },
                            "sexual": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            },
                            "violence": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            }
                        }
                    }
                },
                "required": [
                    "prompt_index",
                    "content_filter_results"
                ]
            }
        },
        "choices": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "finish_details": {
                        "type": "object",
                        "properties": {
                            "type": {
                                "type": "string"
                            },
                            "stop": {
                                "type": "string"
                            }
                        }
                    },
                    "index": {
                        "type": "integer"
                    },
                    "message": {
                        "type": "object",
                        "properties": {
                            "role": {
                                "type": "string"
                            },
                            "content": {
                                "type": "string"
                            }
                        }
                    },
                    "content_filter_results": {
                        "type": "object",
                        "properties": {
                            "hate": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            },
                            "self_harm": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            },
                            "sexual": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            },
                            "violence": {
                                "type": "object",
                                "properties": {
                                    "filtered": {
                                        "type": "boolean"
                                    },
                                    "severity": {
                                        "type": "string"
                                    }
                                }
                            }
                        }
                    }
                },
                "required": [
                    "finish_details",
                    "index",
                    "message",
                    "content_filter_results"
                ]
            }
        },
        "usage": {
            "type": "object",
            "properties": {
                "prompt_tokens": {
                    "type": "integer"
                },
                "completion_tokens": {
                    "type": "integer"
                },
                "total_tokens": {
                    "type": "integer"
                }
            }
        }
    }
}
```

- now loop the output and retrieve the output message and assign to a variable

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-6.jpg 'Mistralai')

- finally send back to power apps

![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-7.jpg 'Mistralai')

### Step 2 - Create the power apps

- First create a blank page
- Add a button and a image control
- Add a text box to hold the response from flow
- Add a text box to hold the prompt


![info](https://github.com/balakreshnan/Samples2024/blob/main/PowerApps/images/gpt4vision-8.jpg 'Mistralai')

- now add the flow created in step 1
- Then configure the onselect of the button to call the flow

```
Set(result,"Please wait while we process your image....");
 
Set(imageJSON,JSON(UploadedImage3.Image,JSONFormat.IncludeBinaryData));
 
Set(ltrim,Right(imageJSON,(Len(imageJSON)-Find("base64,",imageJSON)-6)));
 
Set(fbase64,Left(ltrim,(Len(ltrim)-1)));
 
//call the flow or power automate
Set(result,gpt4visionupd.Run("",fbase64,TextInput22.Text).output)
```

- now assign the result variable to the right side text box to show the output
- Try with different prompts for image and see the output
- Done