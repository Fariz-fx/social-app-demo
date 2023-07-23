# Lesson 10

<- Back to [previous lesson](https://github.com/mongodb-developer/social-app-demo/tree/9-lesson)

---

## Goal

The goal of this lesson is to deploy our completed project to Vercel.

> Be sure to switch to the `9-lesson` branch in your local environment.

## Task 1: Create a search index

1. From the Atlas dashboard, select **Database** from the left menu.
1. Click the **Browse Collections** button on your cluster.
1. Select the **Search** tab.
1. Click the **Create Index** button.
1. Choose the **Visual Editor** and click **Next**.
1. You can name your index or leave it set to `default`.
    - If you alter the name, take note of the new name for later.
1. Choose the `social_butterfly` database and `flutters` collection, then press **Next**.
1. Press **Create Search Index**.

If you don't already have a [Vercel account](https://vercel.com/signup), create one.

## Task 2: Create a Vercel Project

1. From the Vercel dashboard, create a new project.
1. Choose **Continue with GitHub**.
1. Find the **social-app-demo** repository in the list and choose **Import**.
1. Select **Next.js** as the framework.
1. Under **Environment Variables**, add all of your environment variables from your [`.env.local`](./.env.local) file.
1. Click **Deploy**.
    - This initail deploy will not really do anything since the `main` branch does not contain our final code.
1. Go back to the main dashboard and select your new project.
    - Take note of your custom domain. (Example: `https://socialbutterfly.vercel.app`)
1. From the **Settings** tab, click **Git**.
1. Change the **Production Branch** to `10-lesson` and save.
1. Navigate to the **Environment Variables** tab.
1. Change `AUTH0_BASE_URL` to your custom Vercel domain noted earlier. 
1. To redeploy, you'll need to make a new commit to the `10-lesson` branch. You can simply open the `README.md` file, add a space somewhere, and make a new commit.

## Task 3: Update Auth0 settings

The term can be retrieved from the query parameter: `req.query.term`.

The Data API endpoint for this route should be `/aggregate`.

To use the search index, include a `pipeline` in the request body.

> Here is an example pipeline using a search index:
>
> ```js
> [
>   {
>     $search: {
>       index: "default", // The name of the index
>       text: {
>         query: "Hello", // The search term
>         path: {
>           wildcard: "*", // The fields to search
>         },
>       },
>     },
>   },
> ];
> ```

After the search pipeline stage, add a sort stage that sorts descending on the `postedAt` field.

### Test

Test your search functionality by running the application and using the search input in the header.

Notice that the search results are very strict. You have to be exact with the search term.

We can fix this by adding a `fuzzy` parameter to our search pipeline stage.

<details>
<summary>Show solution</summary>

```js
case "GET":
  const term = req.query.term;
  const readData = await fetch(`${baseUrl}/aggregate`, {
    ...fetchOptions,
    body: JSON.stringify({
      ...fetchBody,
      pipeline: [
        {
          $search: {
            index: "default",
            text: {
              query: term,
              path: {
                wildcard: "*",
              },
              fuzzy: {}
            },
          },
        },
        { $sort: { postedAt: -1 } },
      ],
    }),
  });
  const readDataJson = await readData.json();
  res.status(200).json(readDataJson.documents);
  break;
```

</details>

---

Great job! Your application is now deployed and you can open it from the **Overview** tab in Vercel or by navigating to your custom Vercel domain.

> Be sure to commit your branch changes and switch to the `10-lesson` branch in your local environment.
