# lacinia-gen

`lacinia-gen` lets you generate graphs of data using your [lacinia](https://github.com/walmartlabs/lacinia) schema,
so you can make your tests more rigorous.

## Usage

```clojure
(require '[lacinia-gen.core :as lgen])
(require '[clojure.test.check.generators :as g])

(let [schema {:enums {:position {:values [:goalkeeper :defence :attack]}}
              :objects {:team {:fields {:wins {:type 'Int}
                                        :losses {:type 'Int}
                                        :players {:type '(list :player)}}}
                        :player {:fields {:name {:type 'String}
                                          :age {:type 'Int}
                                          :position {:type :position}}}}}]

  (let [gen (lgen/generator schema)]

    (g/sample (gen :position) 5)
    ;; => (:defence :goalkeeper :defence :goalkeeper :goalkeeper)

    (g/sample (gen :player) 5)
    ;; => ({:name "", :age 0, :position :attack}
           {:name "", :age 0, :position :attack}
           {:name "", :age 1, :position :attack}
           {:name "m", :age -2, :position :defence}
           {:name "¤", :age 4, :position :defence})

    (g/sample (gen :team) 5)
    ;; => ({:wins 0, :losses 0, :players ()}
           {:wins 1,
            :losses 1,
            :players ({:name "", :age -1, :position :defence})}
           {:wins 1,
            :losses 2,
            :players
            ({:name "", :age 1, :position :attack}
             {:name "q", :age -2, :position :attack})}
           {:wins -3, :losses 1, :players ()}
           {:wins 0,
            :losses -3,
            :players
            ({:name "ßéÅ", :age 3, :position :attack}
             {:name "", :age -4, :position :defence})})))
```

It supports recursive graphs, so the following works too:

```clojure
(let [schema {:objects {:team {:fields {:name {:type 'String}
                                          :players {:type '(list :player)}}}
                          :player {:fields {:name {:type 'String}
                                            :team {:type :team}}}}}]

    (let [gen (lgen/generator schema)]
      (g/sample (gen :team) 5)))

;; => {:name "ö",
       :players
       ({:name "³2",
         :team
         {:name "ºN",
          :players
          ({:name "Ïâ¦", :team {:name "¤¼J"}}
           {:name "o", :team {:name "æ8"}}
           {:name "/ç", :team {:name "ãL"}}
           {:name "éíª6", :team {:name "v"}})}}
```

## License

Copyright © 2018 oliyh

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
