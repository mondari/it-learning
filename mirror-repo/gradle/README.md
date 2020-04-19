在 {HOME}/.gradle 文件夹下创建 init.gradle 文件，任选以下两种方式之一配置：

方式一：

```groovy
allprojects{
	repositories {
		mavenLocal()
		maven { url 'https://maven.aliyun.com/repository/public/' }
        maven { url 'https://maven.aliyun.com/repository/spring/'}
        maven { url 'https://maven.aliyun.com/repository/spring-plugin/'}
        maven { url 'https://maven.aliyun.com/repository/google/'}
        maven { url 'https://maven.aliyun.com/repository/gradle-plugin/'}
	}
	buildscript {
		repositories {
			mavenLocal()
			maven { url 'https://maven.aliyun.com/repository/public/' }
        	maven { url 'https://maven.aliyun.com/repository/spring/'}
        	maven { url 'https://maven.aliyun.com/repository/spring-plugin/'}
        	maven { url 'https://maven.aliyun.com/repository/google/'}
        	maven { url 'https://maven.aliyun.com/repository/gradle-plugin/'}
		}
	}
}
```

方式二：

```groovy
allprojects {
    repositories {
        // public 是 maven 和 jcenter 的聚合仓库
        def ALIYUN_PUBLIC_URL = 'https://maven.aliyun.com/repository/public/'
        def ALIYUN_MAVEN_URL = 'https://maven.aliyun.com/repository/central/'
        def ALIYUN_JCENTER_URL = 'https://maven.aliyun.com/repository/jcenter/'
        def ALIYUN_GOOGLE_URL = 'https://maven.aliyun.com/repository/google/'
        def ALIYUN_GRADLE_PLUGIN_URL = 'https://maven.aliyun.com/repository/gradle-plugin/'
        all { ArtifactRepository repo ->
            if (repo instanceof MavenArtifactRepository) {
                def url = repo.url.toString()
                if (url.startsWith('https://repo1.maven.org/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_MAVEN_URL."
                    remove repo
                }
                if (url.startsWith('https://jcenter.bintray.com')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_JCENTER_URL."
                    remove repo
                }
                if (url.startsWith('https://dl.google.com/dl/android/maven2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_GOOGLE_URL."
                    remove repo
                }
                if (url.startsWith('https://plugins.gradle.org/m2')) {
                    project.logger.lifecycle "Repository ${repo.url} replaced by $ALIYUN_GRADLE_PLUGIN_URL."
                    remove repo
                }
            }
        }
        // 添加仓库
        mavenLocal()
        maven { url ALIYUN_PUBLIC_URL }
        maven { url ALIYUN_GOOGLE_URL }
        maven { url ALIYUN_GRADLE_PLUGIN_URL }
    }
}
```



 