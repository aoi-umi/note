
删除x天前的job
``` Groovy
import hudson.model.*
def items = Hudson.instance.allItems

def numberOfDays = 30
int count = 0
Date now = new Date()
Date xDaysAgo = new Date(((long)now.time-(1000L*60*60*24*numberOfDays)))

items.each { item ->
  if (item instanceof Job) {
    def builds = item.getBuilds()
    builds.findAll { build -> 
        build.getTime().before(xDaysAgo)
    }.each { build ->
      build.delete()
      count++
    }
  }
}

println "delete ${count}"
```
