title: angularjs利用ui-route异步加载组件
date: 2016/11/6
categories:
- coding
tags:
- javascript
- angularjs
---
ui-route相比于angularjs的原生视图路由更好地支持了路由嵌套，状态转移等等。随着视图不断增加，打包的js体积也会越来越大，比如我在应用里面用到了[wangeditor](https://github.com/wangfupeng1988/wangEditor)里面单独依赖的jquery就300多k。异步加载各个组件就很有必要。在这里我就以ui-route为框架来进行异步加载说明。

首先看一下路由加载文件

```
angular.module('webtrn-sns').config(['$stateProvider', function ($stateProvider) {
    $stateProvider.state({
            name: 'home.message',
            url: '/message',
            abstract: true,
            templateProvider: ['resources', function (resources) {
                return resources.template
            }],
            controllerProvider: ['resources', (resources)=> {
                return resources.controller
            }],
            onEnter: ['resources', (resources)=>resources.css.use()],
            onExit: ['resources', (resources)=>resources.css.unuse()],
            resolve: {
                resources: ()=> {
                    return new Promise(
                        resolve => {
                            require([], () => {
                                resolve({
                                    css: require('./css/message_box.css'),
                                    template: require('./html/message_box.html'),
                                    controller: require('./js/message_box.js')
                                })
                            })
                        }
                    );
                }
            }
        }
    ).state({
            name: 'home.message.add_message',
            url: '/add_message?isReply&toUid&title',
            params: {isReply: null, toUid: null, title: null},
            templateProvider: ['resources', function (resources) {
                return resources.template
            }],
            controllerProvider: ['resources', (resources)=> {
                return resources.controller
            }],
            onEnter: ['resources', (resources)=>resources.css.use()],
            onExit: ['resources', (resources)=>resources.css.unuse()],
            resolve: {
                resources: ()=> {
                    return new Promise(
                        resolve => {
                            require(['./js/message.js'], () => {
                                resolve({
                                    css: require('./css/add_message.css'),
                                    template: require('./html/add_message.html'),
                                    controller: require('./js/add_message.js')
                                })
                            })
                        }
                    );
                }
            }
        }
    )
}])
```
这个是路由状态的一个声明文件，`name`,`url`,`param字段的`方式不变，关键是看resolve这个部分。根据ui-route的[resolve](https://github.com/angular-ui/ui-router/wiki#resolve)文档,resolve是为了给state或者controller进行自定义注入对象的。

下面是举出文档中关于resolve的例子：
````
$stateProvider.state('myState', {
      resolve:{

         // Example using function with simple return value.
         // Since it's not a promise, it resolves immediately.
         simpleObj:  function(){
            return {value: 'simple!'};
         },

         // Example using function with returned promise.
         // This is the typical use case of resolve.
         // You need to inject any services that you are
         // using, e.g. $http in this example
         promiseObj:  function($http){
            // $http returns a promise for the url data
            return $http({method: 'GET', url: '/someUrl'});
         },

         // Another promise example. If you need to do some 
         // processing of the result, use .then, and your 
         // promise is chained in for free. This is another
         // typical use case of resolve.
         promiseObj2:  function($http){
            return $http({method: 'GET', url: '/someUrl'})
               .then (function (data) {
                   return doSomeStuffFirst(data);
               });
         },        

         // Example using a service by name as string.
         // This would look for a 'translations' service
         // within the module and return it.
         // Note: The service could return a promise and
         // it would work just like the example above
         translations: "translations",

         // Example showing injection of service into
         // resolve function. Service then returns a
         // promise. Tip: Inject $stateParams to get
         // access to url parameters.
         translations2: function(translations, $stateParams){
             // Assume that getLang is a service method
             // that uses $http to fetch some translations.
             // Also assume our url was "/:lang/home".
             return translations.getLang($stateParams.lang);
         },

         // Example showing returning of custom made promise
         greeting: function($q, $timeout){
             var deferred = $q.defer();
             $timeout(function() {
                 deferred.resolve('Hello!');
             }, 1000);
             return deferred.promise;
         }
      },

      // The controller waits for every one of the above items to be
      // completely resolved before instantiation. For example, the
      // controller will not instantiate until promiseObj's promise has 
      // been resolved. Then those objects are injected into the controller
      // and available for use.  
      controller: function($scope, simpleObj, promiseObj, promiseObj2, translations, translations2, greeting){
          $scope.simple = simpleObj.value;

          // You can be sure that promiseObj is ready to use!
          $scope.items = promiseObj.data.items;
          $scope.items = promiseObj2.items;

          $scope.title = translations.getLang("english").title;
          $scope.title = translations2.title;

          $scope.greeting = greeting;
      }
   })
````
我们可以看到resolve的对象是支持Promise的。

再回到我们之前的代码`templateProvider`和`controllerProvider`我们注入了resources的模板对象和controller对象，`onEnter`和`onExit`注入了css模块。

如果controller中依赖了服务怎么办的？
````
resolve: {
    resources: ()=> {
        return new Promise(
            resolve => {
                require(['./js/message.js'], () => {
                    resolve({
                        css: require('./css/add_message.css'),
                        template: require('./html/add_message.html'),
                        controller: require('./js/add_message.js')
                    })
                })
            }
        );
    }
}
````
可以在require里面将服务注入，如代码中的`message.js`。而为了将服务进行异步加载我们不能用普通的`.factory`或者`.service`。而需要调用`$provide.factory`或者`$provide.service`

如果采用webpack进行编译打包的话就需要`webpack.optimize.CommonsChunkPlugin`的支持，这样可以对js进行拆分打包，达到异步加载js的目的。
