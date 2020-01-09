---
title: enum优化switch case
author: ziyi
tags:
  - javase
  - enum
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - java
  - enum
comments: true
---

# enum优化switch case
```
public class Outer {
    public static void out(String str) {
        System.out.println(str);
    }
    enum Inner {
        JACK(1, "jack") {
            @Override
            public void handle() {
                out(this.getMessage());
            }
        },
        ANNIE(2, "annie") {
            @Override
            public void handle() {
                out(this.getMessage());
            }
        };
        Integer code;
        String message;

        Inner(Integer code, String message) {
            this.code = code;
            this.message = message;
        }

        public Integer getCode() {
            return code;
        }

        public void setCode(Integer code) {
            this.code = code;
        }

        public String getMessage() {
            return message;
        }

        public void setMessage(String message) {
            this.message = message;
        }

        public abstract void handle();
    }

    public static void main(String[] args) {
        String[] options = new String[] {"JACK", "ANNIE"};
        Arrays.stream(options).forEach((str) -> {
            Inner.valueOf(str).handle();
        });
    }
}
```