エラー文

```
$ yarn serve

yarn run v1.22.4
$ vue-cli-service serve
 INFO  Starting development server...
98% after emitting CopyPlugin

 ERROR  Failed to compile with 2 errors                                                                                                               8:06:31 AM

 error  
 
Template execution failed: ReferenceError: features is not defined

  ReferenceError: features is not defined
  
  - index.html:4 eval
    [.]/[html-webpack-plugin]/lib/loader.js!./public/index.html:4:10
  
  - index.html:7 module.exports
    [.]/[html-webpack-plugin]/lib/loader.js!./public/index.html:7:3
  
  - index.js:284 
    [like_ranking]/[html-webpack-plugin]/index.js:284:18
  
  - task_queues.js:97 processTicksAndRejections
    internal/process/task_queues.js:97:5
```

https://stackoverflow.com/questions/54743760/error-deploy-vuejs-app-in-firebase-hosting
