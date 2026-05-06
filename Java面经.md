能说一下 JVM 类的加载机制是怎样的吗？
简单来说，它是指 **JVM 把描述类的数据从 .class 文件（字节码）加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被 JVM 直接使用的 Java 类型（java.lang.Class 对象）的过程**。
类的生命周期包含七个阶段，其中前五个阶段属于类加载的过程：**加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）**。其中验证、准备、解析统称为**链接（Linking）**。
