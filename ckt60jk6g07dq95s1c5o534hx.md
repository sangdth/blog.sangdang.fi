## My journey of learning back end (1): Setup

This is not a series of tutorials. So there are parts that you might have to work on your own, like, setup ESLint, installing/setup Node on your machine. Luckily you can do it with just a couple of Google searchs.

Reason? I want to learn more. I think it's a curse of being developer :D You always want to be better, know more, make things happens easier, work faster. You name it.

In my current company, Smartly, we work with Facebook's API heavily, and it's not the best API system in the world. And Smartly has a super complex system polling system to handle problems may occur from working with Facebook. I admire it and want to learn more from it. But the complexity of the whole system makes it hard to follow. So I want to re-create a mock version of it, using similar tools, to understand the architecture, how the data flow, etc.

That's why I chose the tech stack as close as possible. Koa.js, Bull, Docker, TypeScript.

First step is creating a working server with Koa. I learned a lot from  [this template](https://github.com/javieraviles/node-typescript-koa-rest) .

This is code in `server.ts`. You can see the whole snapshot in  [this commit](https://github.com/sangdth/canvasser/commit/15f4854b3d84b850f4bfa56df5a1bf55afc8d149) .
````
import Koa from 'koa';
import Router from 'koa-router';

const app = new Koa();
const router = new Router();

router.get('/', async (ctx) => {
  ctx.body = 'Hello World!';
});

app.use(router.routes());

app.listen(3000);

console.log('Server running on port 3000');

````
Note: Some old tutorials might show you the router with `/*` and it will cause the error:
````
/your/path/node_modules/path-to-regexp/src/index.ts:157
    throw new TypeError(`Unexpected ${nextType} at ${index}, expected ${type}`);
          ^
TypeError: Unexpected MODIFIER at 1, expected END
    ...
(/your/path/node_modules/koa-router/lib/router.js:200:12)
   ...
````
According to Koa team, it's a feature, not a bug :D 

![Screenshot 2021-09-04 at 12.43.10.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1630756987559/nJi6o6IDE.png)

OK, server can run, now we need to setup Docker. I will write more details in this section because this is the part I have to learn.

```Dockerfile
# Dockerfile contains specifications for creating an image

ARG NODE_VERSION=14.15.2

# This tells Docker to install an OS (Alpine) with Node
FROM node:${NODE_VERSION}-alpine

# EXPOSE is not publishing any ports, it is just a form of a documentation
EXPOSE 3000 9229

#  Creates a folder in the docker container called 'src'
WORKDIR /src

# Because Alpine is so small that we have to install things we need
RUN apk add	postgresql-client redis

ARG environment

# required for docker:test:watch
RUN if [ "${environment}" = "development" ]; then apk add git; fi

# Copy the dependencies file into our container folder
ADD package.json yarn.lock ./

# This downloads and installs all the dependencies we need for our app
RUN yarn install --frozen-lockfile --link-duplicates

# Copy everything from our current directory to the WORKDIR. This moves all of our source code
ADD . .

RUN yarn build

CMD ["yarn", "start"]
```
The setup copied  [from this article](https://towardsdatascience.com/docker-for-absolute-beginners-what-is-docker-and-how-to-use-it-examples-3d3b11efd830) .


Now, make the `docker-compose.yml`, I learned  [from this post](https://morioh.com/p/b1b47d94f1de).

I want to have a base yml file, and then one for development, and one for production, so I use the `extends`:

### `docker-compose.base.yml`
```yml
version: "3.8"

services:
  redis:
    image: redis:6.2-alpine
    healthcheck:
      test: "[ $$(redis-cli ping) == 'PONG' ]"
      interval: 5s
    ports:
      - "24001:6379"
  postgres:
    image: postgres:13.4-alpine
    ports:
      - "5432:5432"
```

### `docker-compose.local.yml`: (dev)
```yml
version: "3.8"

services:
  redis:
    extends:
      file: docker-compose.base.yml
      service: redis
  postgres:
    extends:
      file: docker-compose.base.yml
      service: postgres
    environment:
      - POSTGRES_DB=canvasser_development
      - POSTGRES_USER=canvasser
      - POSTGRES_HOST_AUTH_METHOD=trust
  app:
    build:
      context: .
      args:
        environment: "development"
    volumes:
      - .:/src
    working_dir: /src
    environment:
      - POSTGRES_URL=postgres://canvasser@postgres:5432/canvasser_development
      - REDIS_URL=redis://redis:6379/14
    depends_on:
      - postgres
      - redis

```

### `docker-compose.yml`
```yml
version: "3.8"

services:
  redis:
    extends:
      file: docker-compose.base.yml
      service: redis
  postgres:
    extends:
      file: docker-compose.base.yml
      service: postgres
  app:
    build: .
    volumes:
      - .:/src
    ports:
      - "19000:19000"
    environment:
      - PORT=19000
```

Now run `docker-compose -f docker-compose.local.yml up` and we will see `postgres`, `redis`, and our app will be up and running.

Next step we will start connect our Koa app into Postgres, and get Bull working.

---

## Some notes:

- ### `ADD` and `COPY`
There are  [some differences between](https://stackoverflow.com/questions/24958140/what-is-the-difference-between-the-copy-and-add-commands-in-a-dockerfile)  `ADD` and `COPY`, but in a nutshell, "the major difference is that ADD can do more than COPY:"

```
- ADD allows <src> to be a URL
- Referring to comments below, the ADD documentation states that:
```
> 
If is a local tar archive in a recognized compression format (identity, gzip, bzip2 or xz) then it is unpacked as a directory. Resources from remote URLs are not decompressed.

- ###  Documentation is not correct
 [This comment](https://github.com/moby/moby/issues/31101#issuecomment-865801157)  in GitHub confirms that we can use `extends` in version 3, but the  [documentation](https://docs.docker.com/compose/extends/#understand-the-extends-configuration)   [still not update](https://github.com/docker/compose/pull/7588#issuecomment-709354500)  (yet).

- ### If you see the error: 
```text
failed to create LLB definition: no build stage in current context
```
Check  [this gist](https://gist.github.com/pich4ya/4942fd2c854044500c90c8297a5ca994) . In short, do not add anything before `FROM` in Dockerfile.