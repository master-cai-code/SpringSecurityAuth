2020/4/30 __  22:25
## 1. 基于角色访问控制、资源访问控制（什么人 干 什么事）
          1.1  角色访问：判断某个用户是否拥有某个角色，根据角色进行控制.硬编码不易扩展 ,适用于角色固定的系统
                        `if (user.hasRole("Project Manager") ) {
                          //show the project report button
                        } else {
                            //don't show the button
                        }`
          1.2  资源访问：直接判断用户是否有操作权限，粒度更细
                             if (user.isPermitted("projectReport:view:12345"))
                             {    //show the project report button    } 
                             else {   
                                 //don't show the button
                             }
                             
                             
 ## 2.基于角色、资源访问控制的实现
   ### 2.1 数据库建表（4+3）
           用户表(users)：id、用户名、密码、状态(是否可用，下同)
           角色表(roles)：id、角色名、角色状态
           权限表(permissions)：id、权限名、权限状态---（可进行的操作）
           资源表（resources）：id、URL、描述
        还有将这几个关联起来的表
           用户角色关联表(users_roles)：用户id、角色id（复合主键）
           角色权限关联表(roles_permissions)：角色id、权限id（复合主键）
           权限资源关联表（permissions_resources）：权限id、资源id（复合主键）
           
   ### 2.2 代码思路
          2.2.1 系统启动的时候，建立一个Map<String,Set> resourceMap，
                键名为resource（即URL），键值为跟该resource关联的所有permission。
                由于一个资源可能对应多个permission（一个资源有多种操作），因此这里需要用到一个集合。
                
          2.2.2 当用户登录的时候，获取该用户所对应的角色，再根据角色获取其拥有的permission集合 
          
          2.2.3 当用户访问受限资源的时候，通过URL从resourceMap中获取所需的permission集合，再跟该用户拥有的permission集合比对，
                用户有相应的permission则通过，否则不通过。
                
  ### 2.3 请求的大致流程：
      filter--->dispatherServlet---->intercepter---->controller
      过滤器     分发器                拦截器           控制器
