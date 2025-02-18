# Build a full-stack react application

## Introduction
this project demonstrates how to create a simple full-stack web application using AWS Amplify. Amplify offers a Git-based CI/CD workflow for building, deploying, and hosting single-page web applications or static sites with serverless backends.

## prerequisites
- AWS accout 
- installed npm and nodejs

## steps to build and deploy on aws amplify
in this step we will start by creating a new React application and pushing it to a GitHub repository. Then, connect the repository to AWS Amplify web hosting and deploy it to a globally available content delivery network (CDN) hosted on an amplifyapp.com domain

### 1. create a new react application 
```sh
npm create vite@latest notesapp -- --template react
cd notesapp
npm install
npm run dev

```

### 2. install the amplify packages
```sh
npm create amplify@latest -y
```

push the project on github

### 3. deploy your app with aws amplify
In this step, we will connect the GitHub repository you just created to AWS Amplify. This will enable you to build, deploy, and host your app on AWS.

1. Sign in to the AWS Management console in a new browser window, and open the AWS Amplify console at https://console.aws.amazon.com/amplify/apps.

2. Choose Create new app.
3. On the Start building with Amplify page, for Deploy your app, select GitHub, and select Next.
4. When prompted, authenticate with GitHub. You will be automatically redirected back to the Amplify console. Choose the repository and main branch you created earlier. Then, select Next.
5. Review the inputs selected, and choose Save and deploy.

AWS Amplify will now build your source code and deploy your app at https://...amplifyapp.com, and on every git push your deployment instance will update. It may take up to 5 minutes to deploy your app.

6. Once the build completes, select the Visit deployed URL button to see your web app up and running live. 

### 4. Initialize  a local amplify 
In this task, we will use AWS Amplify to provision a cloud backend for the app.

1.  set up Amplify auth
By default, our auth resource is configured as shown inside the notesapp/amplify/auth/resource.ts file. For this tutorial, keep the default auth set up as is.

2.  set up Amplify data
- On your local machine, navigate to the notesapp/amplify/data/resource.ts file and update it with the following code. Then, save the file.

The following updated code uses a per-owner authorization rule allow.owner() to restrict the note record’s access to the owner of the record. 
Amplify will automatically add an owner: a.string() field to each note which contains the note owner's identity information upon record creation.

```sh 
import { type ClientSchema, a, defineData } from '@aws-amplify/backend';

const schema = a.schema({
  Note: a
    .model({
      name:a.string(),
      description: a.string(),
      image: a.string(),
    })
    .authorization((allow) => [allow.owner()]),
});

export type Schema = ClientSchema<typeof schema>;

export const data = defineData({
  schema,
  authorizationModes: {
    defaultAuthorizationMode: 'userPool',
  },
});

```

3. set up Amplify storage
we create a new folder  name storage in notesapp/amplify folder, and then create a file named resource.ts inside of the new storage folder.

Update the amplify/storage/resource.ts file with the following code to configure a storage resource for your app. Then, save the file.

The updated code will set up the access so that only the person who uploads the image can access. The code will use the entity_id as a reserved token that will be replaced with the users' identifier when the file is being uploaded. 

``` sh
import { defineStorage } from "@aws-amplify/backend";

export const storage = defineStorage({
  name: "amplifyNotesDrive",
  access: (allow) => ({
    "media/{entity_id}/*": [
      allow.entity("identity").to(["read", "write", "delete"]),
    ],
  }),
});

```

4. update file notesapp/amplify/backend.ts by adding configuration storage 

```sh
import { defineBackend } from '@aws-amplify/backend';
import { auth } from './auth/resource';
import { data } from './data/resource';
import { storage } from './storage/resource';

/**
 * @see https://docs.amplify.aws/react/build-a-backend/ to add storage, functions, and more
 */
defineBackend({
  auth,
  data,
  storage
});
```
### 5. Build the Frontend
In this task, we will create an app frontend and connect it to the cloud backend you have already built.

1. install the amplify libraries

```sh
npm install aws-amplify @aws-amplify/ui-react 
```

finally we implement the style and the application notes in both file notesapp/src/index.css file and notesapp/src/App.jsx like 

