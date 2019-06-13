### extends | mixin  

extends的优先级比mixin高。Vue源码：  

    const extendsFrom = child.extends
    if (extendsFrom) {
        parent = mergeOptions(parent, extendsFrom, vm)
    }
    if (child.mixins) {
        for (let i = 0, l = child.mixins.length; i < l; i++) {
        parent = mergeOptions(parent, child.mixins[i], vm)
        }
    }  

### Demo  

    const Parent1 = {
        created() => {
            console.log('extend parent')
        }
    }

    const parent2 = {
        created() => {
            console.log('mixin parent2')
        }
    }

    const parent3 = {
        created() => {
            console.log('mixin parent3')
        }
    }

    cont child = new Vue.component("child", {
        extends: Parent1,
        mixin: [parent2, parent3],
        created() => {
            console.log('child')
        }
    })

    // console
    // 
    // extend parent
    // mixin parent2
    // mixin parent3
    // child