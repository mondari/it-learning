设置好环境变量 `GRADLE_USER_HOME` 为自己喜欢的目录，默认为 `$HOME/.gradle` 目录。然后将 init.gradle 文件复制到 `$GRADLE_USER_HOME` 目录下。

由于 Gradle 不像 Maven 一样有镜像仓库的概念，所以只能采取替换的方式

init.gradle 文件内容示例：

```groovy
allprojects {
    repositories {
        def MIRROR_PUBLIC_URL = 'https://maven.aliyun.com/repository/public'
        def MIRROR_MAVEN_URL = 'https://maven.aliyun.com/repository/central'
        def MIRROR_JCENTER_URL = 'https://maven.aliyun.com/repository/jcenter'
        def MIRROR_GOOGLE_URL = 'https://maven.aliyun.com/repository/google'
        def MIRROR_GRADLE_PLUGIN_URL = 'https://maven.aliyun.com/repository/gradle-plugin'
        def MIRROR_SPRING_URL = 'https://maven.aliyun.com/repository/spring'
        def MIRROR_SPRING_PLUGIN_URL = 'https://maven.aliyun.com/repository/spring-plugin'
        def MIRROR_GRAILS_CORE_URL = 'https://maven.aliyun.com/repository/grails-core'
        def MIRROR_APACHE_SNAPSHOTS_URL = 'https://maven.aliyun.com/repository/apache-snapshots'
        def ORIGIN_MAVEN_URL = 'https://repo1.maven.org/maven2'
        def ORIGIN_JCENTER_URL = 'http://jcenter.bintray.com'
        def ORIGIN_GOOGLE_URL = 'https://maven.google.com'
        def ORIGIN_GRADLE_PLUGIN_URL = 'https://plugins.gradle.org/m2'
        def ORIGIN_SPRING_URL = 'http://repo.spring.io/libs-milestone'
        def ORIGIN_SPRING_PLUGIN_URL = 'http://repo.spring.io/plugins-release'
        def ORIGIN_GRAILS_CORE_URL = 'https://repo.grails.org/grails/core'
        def ORIGIN_APACHE_SNAPSHOTS_URL = 'https://repository.apache.org/snapshots'
        all { ArtifactRepository repo ->
            if (repo instanceof MavenArtifactRepository) {
                def url = repo.url.toString()
                if (url.startsWith(ORIGIN_MAVEN_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_MAVEN_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_JCENTER_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_JCENTER_URL."
                    remove repo
                } else if (url.startsWith('https://dl.google.com/dl/android/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_GOOGLE_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_GOOGLE_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_GOOGLE_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_GRADLE_PLUGIN_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_GRADLE_PLUGIN_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_SPRING_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_SPRING_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_SPRING_PLUGIN_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_SPRING_PLUGIN_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_GRAILS_CORE_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_GRAILS_CORE_URL."
                    remove repo
                } else if (url.startsWith(ORIGIN_APACHE_SNAPSHOTS_URL)) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $MIRROR_APACHE_SNAPSHOTS_URL."
                    remove repo
                }
            }
        }
        // 添加仓库
        mavenLocal()
        maven { url MIRROR_PUBLIC_URL }
        maven { url MIRROR_MAVEN_URL }
        maven { url MIRROR_JCENTER_URL }
        maven { url MIRROR_GOOGLE_URL }
        maven { url MIRROR_GRADLE_PLUGIN_URL }
        maven { url MIRROR_SPRING_URL }
        maven { url MIRROR_SPRING_PLUGIN_URL }
        maven { url MIRROR_GRAILS_CORE_URL }
        maven { url MIRROR_APACHE_SNAPSHOTS_URL }
    }
}
```



参考：

https://mirrors.huaweicloud.com/

https://maven.aliyun.com/mvn/guide?spm=a2c6h.13651104.0.0.7b4436a4i7zBIK

 