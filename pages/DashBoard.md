# 项目
	- #+BEGIN_QUERY
	  {
	   :title [:h3 "🚀 正在进行中的项目"]
	   :query [:find (pull ?p [*])
	          :where
	          [?p :block/name _]
	          [?p :block/properties ?props]
	          [(get ?props :status) ?s]
	          (not [?p :block/journal? true])]
	   :result-transform (fn [result]
	      (let [target-status "进行中"
	            ;; 1. 提取所有顶级项目（Status匹配且无Parent）
	            parents (->> result
	                         (filter (fn [r]
	                                   (let [props (:block/properties r)
	                                         status (str (get props :status))
	                                         parent (get props :parent)]
	                                     (and (clojure.string/includes? status target-status)
	                                          (or (nil? parent) (= "" (str parent)))))))
	                         (sort-by (fn [r] (str (get-in r [:block/properties :start-date]))))) ]
	        ;; 2. 嵌套子项目（根据parent属性匹配顶级项目的名称）
	        (map (fn [p]
	               {:parent p
	                :children (->> result
	                               (filter (fn [r]
	                                         (let [p-prop (get-in r [:block/properties :parent])
	                                               p-name (if (coll? p-prop) (first p-prop) (str p-prop))]
	                                           (= p-name (:block/original-name p)))))
	                               (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))})
	             parents)))
	   :view (fn [groups]
	      (let [clean (fn [v] (if (coll? v) (first v) (if (or (nil? v) (= "nil" (str v))) "" (str v))))
	            col-name "45%" col-status "15%" col-start "20%" col-end "20%"]
	        [:div {:style {:display "flex" :flex-direction "column" :gap "10px" :padding "10px" :font-family "sans-serif"}}
	         ;; 表头
	         [:div {:style {:display "flex" :padding "0 10px 8px 35px" :opacity "0.6" :font-size "13px" :font-weight "bold" :border-bottom "2px solid #6663"}}
	          [:span {:style {:flex col-name}} "项目/任务名称"]
	          [:span {:style {:flex col-status}} "状态"]
	          [:span {:style {:flex col-start}} "开始日期"]
	          [:span {:style {:flex col-end}} "截止日期"]]
	         
	         (for [g groups]
	           (let [p (:parent g) 
	                 children (:children g) 
	                 p-name (:block/original-name p) 
	                 p-props (:block/properties p)]
	             [:details {:open true :style {:border "1px solid #6662" :border-radius "8px" :overflow "hidden" :box-shadow "0 2px 4px rgba(0,0,0,0.05)"}}
	              ;; 父项目行
	              [:summary {:style {:cursor "pointer" :padding "10px" :background-color "#3b82f61a" :list-style "none"}}
	               [:div {:style {:display "flex" :align-items "center" :width "100%"}}
	                [:span {:style {:flex col-name :font-weight "600" :color "#2563eb"}} "📂 " [:a {:href (str "#/page/" p-name)} p-name]]
	                [:span {:style {:flex col-status}} 
	                 [:span {:style {:font-size "0.8em" :padding "3px 8px" :background-color "#3b82f6" :color "white" :border-radius "12px"}} (clean (get p-props :status))]]
	                [:span {:style {:flex col-start :font-size "0.85em"}} (clean (get p-props :start-date))]
	                [:span {:style {:flex col-end :font-size "0.85em"}} (clean (get p-props :end-date))]]]
	              
	              ;; 子项目列表
	              (if (not-empty children)
	                [:div {:style {:background-color "#ffffff05"}}
	                  (for [c children]
	                    (let [c-name (:block/original-name c) c-props (:block/properties c)]
	                      [:div {:style {:display "flex" :align-items "center" :padding "8px 10px" :border-top "1px solid #6661"}}
	                       [:span {:style {:flex col-name :padding-left "30px" :opacity "0.9"}} 
	                            [:span {:style {:opacity "0.4" :margin-right "10px"}} "└─"]
	                            [:a {:href (str "#/page/" c-name)} c-name]]
	                       [:span {:style {:flex col-status}} 
	                        [:span {:style {:font-size "0.8em" :padding "1px 6px" :border "1px solid #6664" :border-radius "4px"}} (clean (get c-props :status))]]
	                       [:span {:style {:flex col-start :font-size "0.85em" :opacity "0.7"}} (clean (get c-props :start-date))]
	                       [:span {:style {:flex col-end :font-size "0.85em" :opacity "0.7"}} (clean (get c-props :end-date))]]))] 
	                nil)]))]))
	  }
	  #+END_QUERY
	- #+BEGIN_QUERY
	  {
	   :title [:h3 "🔘 待安排项目"]
	   :query [:find (pull ?p [*]) :where [?p :block/properties ?props] [(get ?props :status) ?s] (or [(clojure.string/includes? ?s "待安排")] [(= ?s "待安排")]) (not [?p :block/journal? true])]
	   :result-transform (fn [result]
	      (let [target "待安排"
	            parents (->> result (filter (fn [r] (let [ps (:block/properties r)] (and (clojure.string/includes? (str (get ps :status)) target) (or (nil? (get ps :parent)) (= "" (str (get ps :parent)))))))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))]
	        (map (fn [p] {:parent p :children (->> result (filter (fn [r] (= (if (coll? (get-in r [:block/properties :parent])) (first (get-in r [:block/properties :parent])) (str (get-in r [:block/properties :parent]))) (:block/original-name p)))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))}) parents)))
	   :view (fn [groups] (let [clean (fn [v] (if (coll? v) (first v) (if (or (nil? v) (= "nil" (str v))) "" (str v)))) col-name "45%" col-status "15%" col-start "20%" col-end "20%"]
	        [:div {:style {:display "flex" :flex-direction "column" :gap "8px"}}
	         (for [g groups] (let [p (:parent g) children (:children g) p-name (:block/original-name p) theme "#94a3b8"]
	             [:details {:open true :style {:border (str "1px solid " theme "44") :border-radius "8px" :background-color (str theme "11")}}
	              [:summary {:style {:cursor "pointer" :padding "8px 12px" :list-style "none" :border-left (str "4px solid " theme)}}
	               [:div {:style {:display "flex" :align-items "center" :width "100%"}}
	                [:span {:style {:flex col-name :font-weight "bold"}} [:a {:href (str "#/page/" p-name)} p-name]]
	                [:span {:style {:flex col-status}} [:span {:style {:font-size "0.75em" :padding "2px 8px" :background-color theme :color "white" :border-radius "4px"}} "待安排"]]
	                [:span {:style {:flex col-start :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :start-date]))]
	                [:span {:style {:flex col-end :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :end-date]))]]]
	              (if (not-empty children) [:div {:style {:padding "2px 0" :border-top "1px solid #6661"}} (for [c children] [:div {:style {:display "flex" :padding "6px 12px" :font-size "0.95em"}} [:span {:style {:flex col-name :padding-left "24px" :opacity "0.8"}} "┕ " [:a {:href (str "#/page/" (:block/original-name c))} (:block/original-name c)]] [:span {:style {:flex col-status}} [:span {:style {:font-size "0.8em" :color theme}} (clean (get-in c [:block/properties :status]))]] [:span {:style {:flex col-start :opacity "0.5"}} (clean (get-in c [:block/properties :start-date]))] [:span {:style {:flex col-end :opacity "0.5"}} (clean (get-in c [:block/properties :end-date]))]])] nil)]))]))
	  }
	  #+END_QUERY
	- #+BEGIN_QUERY
	  {
	   :title [:h3 "⏳ 待开始项目"]
	   :query [:find (pull ?p [*]) :where [?p :block/properties ?props] [(get ?props :status) ?s] (or [(clojure.string/includes? ?s "待开始")] [(= ?s "待开始")]) (not [?p :block/journal? true])]
	   :result-transform (fn [result]
	      (let [target "待开始"
	            parents (->> result (filter (fn [r] (let [ps (:block/properties r)] (and (clojure.string/includes? (str (get ps :status)) target) (or (nil? (get ps :parent)) (= "" (str (get ps :parent)))))))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))]
	        (map (fn [p] {:parent p :children (->> result (filter (fn [r] (= (if (coll? (get-in r [:block/properties :parent])) (first (get-in r [:block/properties :parent])) (str (get-in r [:block/properties :parent]))) (:block/original-name p)))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))}) parents)))
	   :view (fn [groups] (let [clean (fn [v] (if (coll? v) (first v) (if (or (nil? v) (= "nil" (str v))) "" (str v)))) col-name "45%" col-status "15%" col-start "20%" col-end "20%"]
	        [:div {:style {:display "flex" :flex-direction "column" :gap "8px"}}
	         (for [g groups] (let [p (:parent g) children (:children g) p-name (:block/original-name p) theme "#f59e0b"]
	             [:details {:open true :style {:border (str "1px solid " theme "44") :border-radius "8px" :background-color (str theme "11")}}
	              [:summary {:style {:cursor "pointer" :padding "8px 12px" :list-style "none" :border-left (str "4px solid " theme)}}
	               [:div {:style {:display "flex" :align-items "center" :width "100%"}}
	                [:span {:style {:flex col-name :font-weight "bold"}} [:a {:href (str "#/page/" p-name)} p-name]]
	                [:span {:style {:flex col-status}} [:span {:style {:font-size "0.75em" :padding "2px 8px" :background-color theme :color "white" :border-radius "4px"}} "待开始"]]
	                [:span {:style {:flex col-start :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :start-date]))]
	                [:span {:style {:flex col-end :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :end-date]))]]]
	              (if (not-empty children) [:div {:style {:padding "2px 0" :border-top "1px solid #6661"}} (for [c children] [:div {:style {:display "flex" :padding "6px 12px" :font-size "0.95em"}} [:span {:style {:flex col-name :padding-left "24px" :opacity "0.8"}} "┕ " [:a {:href (str "#/page/" (:block/original-name c))} (:block/original-name c)]] [:span {:style {:flex col-status}} [:span {:style {:font-size "0.8em" :color theme}} (clean (get-in c [:block/properties :status]))]] [:span {:style {:flex col-start :opacity "0.5"}} (clean (get-in c [:block/properties :start-date]))] [:span {:style {:flex col-end :opacity "0.5"}} (clean (get-in c [:block/properties :end-date]))]])] nil)]))]))
	  }
	  #+END_QUERY
	- #+BEGIN_QUERY
	  {
	   :title [:h3 "📝 待整理项目"]
	   :query [:find (pull ?p [*]) :where [?p :block/properties ?props] [(get ?props :status) ?s] (or [(clojure.string/includes? ?s "待整理")] [(= ?s "待整理")]) (not [?p :block/journal? true])]
	   :result-transform (fn [result]
	      (let [target "待整理"
	            parents (->> result (filter (fn [r] (let [ps (:block/properties r)] (and (clojure.string/includes? (str (get ps :status)) target) (or (nil? (get ps :parent)) (= "" (str (get ps :parent)))))))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))]
	        (map (fn [p] {:parent p :children (->> result (filter (fn [r] (= (if (coll? (get-in r [:block/properties :parent])) (first (get-in r [:block/properties :parent])) (str (get-in r [:block/properties :parent]))) (:block/original-name p)))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))}) parents)))
	   :view (fn [groups] (let [clean (fn [v] (if (coll? v) (first v) (if (or (nil? v) (= "nil" (str v))) "" (str v)))) col-name "45%" col-status "15%" col-start "20%" col-end "20%"]
	        [:div {:style {:display "flex" :flex-direction "column" :gap "8px"}}
	         (for [g groups] (let [p (:parent g) children (:children g) p-name (:block/original-name p) theme "#8b5cf6"]
	             [:details {:open true :style {:border (str "1px solid " theme "44") :border-radius "8px" :background-color (str theme "11")}}
	              [:summary {:style {:cursor "pointer" :padding "8px 12px" :list-style "none" :border-left (str "4px solid " theme)}}
	               [:div {:style {:display "flex" :align-items "center" :width "100%"}}
	                [:span {:style {:flex col-name :font-weight "bold"}} [:a {:href (str "#/page/" p-name)} p-name]]
	                [:span {:style {:flex col-status}} [:span {:style {:font-size "0.75em" :padding "2px 8px" :background-color theme :color "white" :border-radius "4px"}} "待整理"]]
	                [:span {:style {:flex col-start :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :start-date]))]
	                [:span {:style {:flex col-end :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :end-date]))]]]
	              (if (not-empty children) [:div {:style {:padding "2px 0" :border-top "1px solid #6661"}} (for [c children] [:div {:style {:display "flex" :padding "6px 12px" :font-size "0.95em"}} [:span {:style {:flex col-name :padding-left "24px" :opacity "0.8"}} "┕ " [:a {:href (str "#/page/" (:block/original-name c))} (:block/original-name c)]] [:span {:style {:flex col-status}} [:span {:style {:font-size "0.8em" :color theme}} (clean (get-in c [:block/properties :status]))]] [:span {:style {:flex col-start :opacity "0.5"}} (clean (get-in c [:block/properties :start-date]))] [:span {:style {:flex col-end :opacity "0.5"}} (clean (get-in c [:block/properties :end-date]))]])] nil)]))]))
	  }
	  #+END_QUERY
	- #+BEGIN_QUERY
	  {
	   :title [:h3 "✅ 已归档项目"]
	   :query [:find (pull ?p [*]) :where [?p :block/properties ?props] [(get ?props :status) ?s] (or [(clojure.string/includes? ?s "已归档")] [(= ?s "已归档")]) (not [?p :block/journal? true])]
	   :result-transform (fn [result]
	      (let [target "已归档"
	            parents (->> result (filter (fn [r] (let [ps (:block/properties r)] (and (clojure.string/includes? (str (get ps :status)) target) (or (nil? (get ps :parent)) (= "" (str (get ps :parent)))))))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))]
	        (map (fn [p] {:parent p :children (->> result (filter (fn [r] (= (if (coll? (get-in r [:block/properties :parent])) (first (get-in r [:block/properties :parent])) (str (get-in r [:block/properties :parent]))) (:block/original-name p)))) (sort-by (fn [r] (str (get-in r [:block/properties :start-date])))))}) parents)))
	   :view (fn [groups] (let [clean (fn [v] (if (coll? v) (first v) (if (or (nil? v) (= "nil" (str v))) "" (str v)))) col-name "45%" col-status "15%" col-start "20%" col-end "20%"]
	        [:div {:style {:display "flex" :flex-direction "column" :gap "8px"}}
	         (for [g groups] (let [p (:parent g) children (:children g) p-name (:block/original-name p) theme "#10b981"]
	             [:details {:open true :style {:border (str "1px solid " theme "44") :border-radius "8px" :background-color (str theme "11")}}
	              [:summary {:style {:cursor "pointer" :padding "8px 12px" :list-style "none" :border-left (str "4px solid " theme)}}
	               [:div {:style {:display "flex" :align-items "center" :width "100%"}}
	                [:span {:style {:flex col-name :font-weight "bold"}} [:a {:href (str "#/page/" p-name)} p-name]]
	                [:span {:style {:flex col-status}} [:span {:style {:font-size "0.75em" :padding "2px 8px" :background-color theme :color "white" :border-radius "4px"}} "已归档"]]
	                [:span {:style {:flex col-start :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :start-date]))]
	                [:span {:style {:flex col-end :font-size "0.85em" :opacity "0.7"}} (clean (get-in p [:block/properties :end-date]))]]]
	              (if (not-empty children) [:div {:style {:padding "2px 0" :border-top "1px solid #6661"}} (for [c children] [:div {:style {:display "flex" :padding "6px 12px" :font-size "0.95em"}} [:span {:style {:flex col-name :padding-left "24px" :opacity "0.8"}} "┕ " [:a {:href (str "#/page/" (:block/original-name c))} (:block/original-name c)]] [:span {:style {:flex col-status}} [:span {:style {:font-size "0.8em" :color theme}} (clean (get-in c [:block/properties :status]))]] [:span {:style {:flex col-start :opacity "0.5"}} (clean (get-in c [:block/properties :start-date]))] [:span {:style {:flex col-end :opacity "0.5"}} (clean (get-in c [:block/properties :end-date]))]])] nil)]))]))
	  }
	  #+END_QUERY
-
- # 年计划
	- ## 进行中 -2026 完美剧本
		- ### 总目标：
			- 事业：
			  logseq.order-list-type:: number
				- 找到暑期实习 -》找到SSP
				  logseq.order-list-type:: number
			- 学习：
			  logseq.order-list-type:: number
				- 提前把FYP等弄完，马校丝滑毕业 -》提前把材料交齐，宿舍找好，研究生丝滑入学 -〉拿到全额奖学金
				  logseq.order-list-type:: number
			- 生活：
			  logseq.order-list-type:: number
				- 看香港身份，做好未来规划，见WZJ爹妈
				  logseq.order-list-type:: number
		- ### 每月安排：
			-
			- 1月：
			- 2月：
			- 3月：
			- 大四下：4.3 - 7.31 大四下
			- 4月：
				- ((67f53943-2a97-4135-9588-391f3bbd67fc))
			- 5月：
			- 6月：
			- 7月：
			- 8月：
			- 9月：
			- 10月：
			- 11月：
			- 12月：
	- ## 我的大好年华
		- ### 2025完美剧本
			- 总：一共有53个week
				- 实习
				- 申研
				- 了解考公
			- 分月：
				- 已结束
				  collapsed:: true
					- 一月
					  background-color:: red
					- 小学期：
					  collapsed:: true
						- 二月：
						  background-color:: red
							- [[GRE]] 考完
							- [[算法]]积累
						- 三月：
						  background-color:: red
						  collapsed:: true
							- 总：
								- 23-29
								  logseq.order-list-type:: number
									- 日本旅游！！！！！
								- 准备面试，正好三月HC多，速速准备然后面一波
								  logseq.order-list-type:: number
									- 3.11-3.18 期间，过完两个（mysql和计网），看看每天的速度是什么水平
									  logseq.order-list-type:: number
										- 更改计划，看12号和13号能不能过完一个，能的话就改改简历去投
										  logseq.order-list-type:: number
								- 继续准备GRE，可以先不刷题把时间留给准备面试和考试，但是单词必须背着走，二十天，每天50个都1000词，但是List还有25个，每天100个合适
								  logseq.order-list-type:: number
									- 看看背50个要花多久，决定一下日本的时候要背多少，等他们化妆、洗澡，肯定有时间的
									  logseq.order-list-type:: number
							- 分：
							  collapsed:: true
								- 3.10-3.16:全年W11，小学期W5
									- 3.11-3.18 期间，过完两个（mysql和计网），看看每天的速度是什么水平
									  logseq.order-list-type:: number
									- DONE 把服务器配置好
									  logseq.order-list-type:: number
										- 各种token找好
										  logseq.order-list-type:: number
										- 小网页写出来
										  logseq.order-list-type:: number
									- DONE 把手机相册清空，去日本拍拍拍！
									  logseq.order-list-type:: number
									  :LOGBOOK:
									  CLOCK: [2025-03-30 Sun 17:33:37]--[2025-03-30 Sun 17:33:38] =>  00:00:01
									  :END:
									- ((67d2b862-a922-4e4d-82e6-f7da6f432f38))
									  logseq.order-list-type:: number
									- DONE 读懂VPN配置文件和原理，把他迁移到我自己服务器上
									  logseq.order-list-type:: number
								- 3.17-3.23:全年W12，小学期考试加休假
								- 3.24-3.30:全年W13，在日本
					- 大三下：15个Week
					  collapsed:: true
						- 四月：
						  background-color:: red
							- 总：
								- 再考一次GRE，刷到425
								- 找实习
								- DONE 提前准备FYP导师，看要不要准备AI相关内容
							- 分：
								- 3.31-4.6:全年W14，
									- ((67e92946-94fc-40df-81e5-80b59dcc3138))
									- ((67e9298f-ce8d-4976-b1f1-c196ac76e7dc))
								- 4.7-4.13:全年W15
									- GRE训练
									- 这周把简历上的过完
									- 算法保持训练
									- LJK的那个确定改进方向
									- LHY的代码看完
								- 4.14-4.20:全年W16
									- DONE 简历过完
										- OSS的看代码
										- DONE AI翻译相关的实现一下
										  :LOGBOOK:
										  CLOCK: [2025-05-12 Mon 16:40:10]--[2025-07-17 Thu 22:35:15] =>  1589:55:05
										  :END:
									- DONE 简历看完之后，GRE和算法同步展开，然后每天过面经和约面
									- DONE 周末开始做ASS
								- 4.21-4.27:全年W17，学期W4
									- 5天每天一个把代码随想录刷完
									- 周末刷hot100
									- br梳理出来的八股漏洞按照 什么情况这样做，怎么做，有什么优缺点，缺点怎么解决的框架去梳理
									- 找实习还差的八股：
										- rabbitMq
										- DONE 计网
										- xxl-job
										- 注册中心（看一遍配置视频）
						- 五月：找到实习
						  background-color:: red
						  collapsed:: true
							- 总：
								- 大事：
									- 再考一次GRE，刷到425
										- DONE 开始背单词
									- DONE 找到实习
									- DONE 提前准备FYP导师，看要不要准备AI相关内容
							- 分：
								- 5.5-5.11 学期W6，全年W18
									- 周一：5号
										- DONE 算法DP（晚上）
										- DONE 计网背诵
										- DONE 八股
									- 周二：6号
										- DONE 算法 图论矩阵贪心
										- DONE 面试
								- 5.12-5.18 学期W7，全年W19
								- 5.19-5.25 学期W8，全年W20
								- 5.26-6.1 学期W9，全年W21
						- 六月：考完GRE 。结果，并没有考出来，在做小组作业
						  background-color:: red
						  collapsed:: true
							- 总：
								- DONE 准备考GRE
								- DONE 制作文书
								- DONE 6月入职之后看节奏订考试
								- DONE 银行开户
								- DONE 交护照续签
								- DONE 七月内把简历写出来，然后给老师发CV要推荐信
								- 分：
								  collapsed:: true
									- 6.2-6.8 学期W10，全年W22
									- 6.9-6.15 学期W11，入职W1，全年W23
									- 6.16-6.22 学期W12，入职W2，全年W24
										- 我艹想想我这周末把这些干完了有多爽：
											- DONE GRE开始安排背单词和刷题
												- DONE 看之前的教训和刷题计划，整理每天刷题的内容 - 1h
												- DONE 看考试时间，定考试时间 - 30min
													- 要考虑到可能考不过
											- DONE 整理算法每天的刷题计划 - 30min
											- DONE 整理八股每天的复习计划表 - 30min s
											- DONE 写完SADE的Assignment - 5h
											  :LOGBOOK:
											  CLOCK: [2025-06-27 Fri 18:23:22]--[2025-07-01 Tue 07:56:07] =>  85:32:45
											  :END:
											- DONE 写计网的ASS
									- 6.23-6.29 学期W13，入职W3，全年W25
						- 七月：考试。结果：CGPA3.67
						  background-color:: red
						  collapsed:: true
							- 总：
								- DONE 给导师发CV，拿推荐信
								- DONE 再考一次雅思
								- DONE 看申研提前批
								- DONE GRE考出来
								- DONE 开始投日常实习练手
							- 6.30-7.6 学期W14，入职W5，全年W26
							- 7.7-7.13 学期W15，入职W6，全年W27
							- 7.14-7.20 学期W16，入职W7，全年W28
							- 6.21-7.27 学期W17（考试周，学期结束），入职W8，全年W29
					- 暑假
					  collapsed:: true
						- 八月：考GRE。结果：写文书，没有弄出来
						  background-color:: red
							- 大三下4.0！！！CGPA：3.73
							- 和wzj一起实习！！！！！
							- 旅游！！！回家好好玩十天！！
						- 九月：考GRE。结果：差两分
						  background-color:: red
						  collapsed:: true
							- 整理国考省考报名信息并准备
							- 投递申研
							- 和wzj一起暑期实习！
					- 大四上：
					  collapsed:: true
						- 十月：找实习，准备暑期实习面试
						  background-color:: red
							- 投递申研
							- 第一段日常实习
							- 第一周，1-5号：考GRE
							- 第二周，6-12号：准备面试 实际：放松
							- 第三周，13-19号：投面试：准备面试
							  collapsed:: true
								- 15号：
									- 八股：
										- 并发（60）
									- 算法：
										- 半月计划第一天和第二天
									- 业务：
										- 把ticket那个handler理解清楚
										- session和conversation那几个生命周期梳理出来
									- 回顾与总结：
										- 算法证明是能做到的
										- 地铁上必须看八股
								- 16号：
									- 八股：
										- 并发（60）
									- 算法：
										- 一月计划第三天到第六天
									- 业务：
										- ticket的看完
								- 18号：
									- 八股：
										- 操作系统
								- 19号：
									- 八股：
										- 计网
							- 第四周，19-26号：投面试：看offer
							- 第五周，27-31号：投面试
						- 十一月：找到实习啦，准备实习
						  collapsed:: true
							- 17号-23号
								- 任务：
									- DONE 找到房子
									- DONE 整理回去十五顿吃什么
									- DONE 整理眉山吃什么
									- DONE 发快递到上海
								- DONE 买衣服，不然去眉山没穿的
									- 买眉山穿的就行，其他的直接发上海，不然干不了
									- DONE 买皮带
							- 23号-30号
								- Task：
									- 去上海之前：
										- LATER 规划明年时间
										  SCHEDULED: <2026-02-01 Sun>
										- 学校：
										- 生活：
											- DONE 银行卡解限额
											- DONE 选睡衣
											- DONE vpn搞定
									- 周三和周四
										- DONE 发邮件给老师，实习
				- # 十二月：认真实习，准备暑期实习面试
					- ## 总目标：
						- 事业：
							- 边实习边写简历，用做好随时跑路的态度来准备 #Q0-Routine
							- 循环刷算法，刷到能背下来 #Q0-Routine
						- 生活：
							- 健身，把肚子减下去 #Q0-Routine
							- 多把精力关注在wzj身上，每周买个礼物给个惊喜 
							  start-date::  [[Dec 22nd, 2025]]
							  deadline::   [[Feb 8th, 2026]]
					- ## W1: 1-7
						- 本周要做：
							- 规划：
								- LATER 规划明年时间 [[2026-01-25]] #Q2-重要不紧急
								- TODO 整理这段时间的教训和下一段实习的随时离职简历书写方案和平时积累，准备周五入职 [[2026-01-25]]
							- 生活：
								- DONE 搞一身好看的行头
								- DONE 研究保湿的脸上用品
								- DONE 规划去上海玩的时间和地图 #Q3-紧急不重要
								  :LOGBOOK:
								  CLOCK: [2025-11-30 Sun 21:54:19]--[2025-11-30 Sun 21:54:19] =>  00:00:00
								  :END:
							- DONE 规划去上海玩的时间和地图
							  :LOGBOOK:
							  CLOCK: [2025-11-30 Sun 21:54:19]--[2025-11-30 Sun 21:54:19] =>  00:00:00
							  :END:
								- 搞一个旅游地图
								- 旅游
									- 不眠之夜
									- 屋有岛密室
								- 打卡
								- 美食
							- 日常：
								- DONE 买mini
								- DONE 申NUS和南洋
								- DONE 买耳机
							- 其他：
								- DONE 域名找个便宜的办法续费了
						- 时间安排：
							- 工作日：
								- 周一：
									- 日常：
										- DONE 搞一身好看的行头
											- DONE 去优衣库买内衣和袜子
											- 积攒穿搭灵感，看中间搭什么
										- DONE 测去公司的时间和通勤价格
									- 抢劵，买东西
										- DONE 买mini
										- DONE 申NUS和南洋
								- 周二：
								- 周三：
								- 周四：
								- 周五：
							- 周末：
								- 周六：
									- DONE 申NUS和南洋
									- DONE 域名找个便宜的办法续费了
									- DONE 规划去上海玩的时间和地图
									  :LOGBOOK:
									  CLOCK: [2025-11-30 Sun 21:54:19]--[2025-11-30 Sun 21:54:19] =>  00:00:00
									  :END:
									- DONE 看时间安排，去俄罗斯玩
					- 9-14
					- 15-21
					- 22-28