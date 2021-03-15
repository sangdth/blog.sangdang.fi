## My two cents opinion in hooks organising

After doing with couple of medium/big projects, I would like to share a little comment on hooks organising.

In my project, usually I try to import as much as possible the hooks from 3rd parties and my custom hooks in one place, so in the future, when any of them upgrade/change, it's easier for me to refactor.

Says, I have a place like this:
```
// my-project/lib/helpers.ts
export { useQuery, useMutation } from '@apollo/client';
export { default as useFoo } from 'foo-project';
export { useBar } from '../myCustomHooks';
```

After that, in 999+ places in my project, I simply get them by :
```
import { ...something I need } from 'lib/helpers';
```
If at anytime the Apollo changed their package name or you want to do your own `useFoo`, you can simply go to only one place and reference it to the new destination. Not going to find and do it in 999+ places.
