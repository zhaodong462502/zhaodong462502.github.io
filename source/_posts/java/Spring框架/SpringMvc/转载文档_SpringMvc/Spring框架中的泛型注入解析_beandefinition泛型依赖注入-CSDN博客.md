## [Spring框架](https://so.csdn.net/so/search?q=Spring%E6%A1%86%E6%9E%B6&spm=1001.2101.3001.7020)中的泛型注入解析

在Spring框架的4.x版本中，引入了一项非常实用的功能，即Java泛型类型作为隐式限定符用于[依赖注入](https://so.csdn.net/so/search?q=%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5&spm=1001.2101.3001.7020)。这项功能极大地简化了依赖注入的复杂性，尤其是在处理具有相同接口或父类但泛型类型不同的bean时。本文将通过一个具体的示例，详细解析Spring框架如何利用泛型类型进行精确的依赖注入。

### 背景知识

在之前的Spring版本中，如果存在多个相同类型的bean，但泛型类型不同，Spring容器在进行自动装配时可能会抛出`NoUniqueBeanDefinitionException`异常，因为没有足够的信息来确定注入哪一个bean。为了解决这个问题，开发者不得不使用`@Qualifier`注解或定义特定的限定类型。

### 泛型注入示例

以下示例展示了如何使用Spring 4.x的泛型注入功能。我们定义了一个`RateFormatter`类，它是一个泛型类，能够根据传入的数字类型格式化数字。

```
public class RateFormatter&lt;T extends Number&gt; {
    public String format(T number){
            NumberFormat format = NumberFormat.getInstance();
                    if(number instanceof Integer){
                                format.setMinimumIntegerDigits(0);
                                        } else if(number instanceof BigDecimal){
                                                    format.setMinimumIntegerDigits(2);
                                                                format.setMaximumFractionDigits(2);
                                                                        }
                                                                                return format.format(number);
                                                                                    }
                                                                                    }
                                                                                    
```

接着，我们定义了一个`RateCalculator`类，它依赖于`RateFormatter<BigDecimal>`类型的bean。

```
public class RateCalculator {
                                                                                        @Autowired
                                                                                            private RateFormatter&lt;BigDecimal&gt; formatter;
    public void calculate() {
            BigDecimal rate = new BigDecimal("1053.75356");
                    System.out.println(formatter.format(rate));
                        }
                        }
                        
```

### 配置与运行示例

在配置类中，我们定义了两个`RateFormatter`的实例，一个用于`Integer`类型，另一个用于`BigDecimal`类型。同时，我们定义了`RateCalculator`的bean，并在`main`方法中启动Spring应用上下文，执行计算。

```
@Configuration
                        public class Config {
                            @Bean
                                RateFormatter&lt;Integer&gt; integerRateFormatter() {
                                        return new RateFormatter&lt;&gt;();
                                            }
    @Bean
        RateFormatter&lt;BigDecimal&gt; bigDecimalRateFormatter() {
                return new RateFormatter&lt;&gt;();
                    }
    @Bean
        RateCalculator rateCalculator() {
                return new RateCalculator();
                    }
    public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
                    RateCalculator bean = context.getBean(RateCalculator.class);
                            bean.calculate();
                                }
                                }
                                
```

### 输出结果

执行上述代码后，控制台将输出格式化后的利率值：

```
1,053.75
                                
```

### 技术栈与依赖

本示例使用了以下技术栈和依赖：

-   Spring Context: `spring-context` 6.1.2
-   兼容的Spring版本：4.0.0.RELEASE - 6.1.2
-   兼容的Java版本：JDK 17+
-   构建工具：Maven 3.8.1

### 结论

通过这个示例，我们可以看到Spring 4.x版本中泛型注入的强大功能。它不仅提高了代码的可读性和可维护性，还减少了配置的复杂度。在处理具有泛型参数的bean时，Spring能够智能地根据泛型类型进行匹配和注入，从而避免了潜在的`NoUniqueBeanDefinitionException`异常。
