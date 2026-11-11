# שיעור 5 — שיעורי בית

## חלק א׳ — מפת ה־Cluster

| רכיב | איפה הוא רץ (הנהלה / כל Node) | מה הוא עושה? | אנלוגיה |
|---|---|---|---|
| API Server | 	Control Plane|מקבל ומאמת בקשות לניהול ה־Cluster | מוקד ההזמנות|
| etcd | Control Plane| מסד נתונים ששומר את מצב ה־Cluster|ספר הזמנות |
| Scheduler |Control Plane |בוחר Node מתאים לכל POD | 	סדרנית עבודה|
| Controller Manager | 	Control Plane|מפעיל בקרים שדואגים שהמצב בפועל יתאים למצב הרצוי |מפקחת |
| kubelet |	כל Node| 	שואלת "מה שובץ אליי?", מריצה, ומדווחת כל כמה שניות שה-node חי| אחראית המשמרת|
| Container Runtime |	כל Node |	מפעיל ומנהל את הקונטיינרים בפועל |התנור |
| kube-proxy | 	בכל Worker Node|	מגדיר כללי רשת לניתוב בקשות מ־Services ל־Pods |שליח |
| CoreDNS |	כ־Pod בתוך ה־Cluster |מתרגם שם של Service לכתובת IP | ספר טלפונים|

## חלק ב׳ — תרחישים
#1
API Server – מקבל את הבקשה לשנות את מספר ה־replicas מ־2 ל־4.

etcd – שומר את המצב הרצוי החדש: 4 Pods.

Controller Manager – מזהה שחסרים שני Pods ודואג ליצירתם באמצעות ה־ReplicaSet.

Scheduler – בוחר Node מתאים לכל אחד מה־Pods החדשים.

kubelet – מזהה את ה־Pods שהוקצו ל־Node שלו ודואג להפעלתם.

Container Runtime – יוצר ומפעיל בפועל את הקונטיינרים.

בסיום, ה־kubelet מדווח על מצב ה־Pods, וכעת יש 4 Pods במצב Running.
#2
1. אם ה־Container של ה־backend קרס: ה־kubelet מזהה את הקריסה ודואג להפעיל מחדש את ה־Container באמצעות ה־Container Runtime, בדרך כלל בתוך אותו Pod.

2. אם מחקו את ה־Pod כולו: ה־Controller Manager (באמצעות ReplicaSet) מזהה שחסר Pod כדי לשמור על מספר העותקים הרצוי, ודואג ליצור Pod חדש. לאחר מכן ה־Scheduler משבץ אותו ל־Node וה־kubelet דואג להפעלתו.
#3
- בענן: אם Node אחד נופל, Kubernetes יכול ליצור Pods חלופיים על Nodes אחרים, אם יש מספיק משאבים.
- בכיתה: אם המחשב נכבה, כל ה־Cluster מפסיק לפעול עד שמדליקים אותו מחדש.
#4
1. **imagePullPolicy: Never** – לא מתאים ל־EKS, כי ה־Image שבנינו בכיתה נמצא במחשב המקומי ולא ב־Nodes בענן. צריך להעלות אותו ל־Registry, לעדכן את כתובת ה־Image ולשנות ל־`IfNotPresent`.
2. **NodePort** – לא מאפשר גישה דרך `localhost` כמו בכיתה. כדי לחשוף את האפליקציה באינטרנט, נשנה את ה־Service ל־`LoadBalancer`.
#5
Pod שנוצר באמצעות `kubectl run` הוא Pod בודד שאינו מנוהל על ידי Deployment, ולכן כשמוחקים אותו הוא לא נוצר מחדש.

לעומת זאת, ה־Pod של `web` מנוהל על ידי Deployment, שמגדיר באמצעות `replicas` כמה Pods צריכים לרוץ. כאשר מוחקים אחד מהם, Kubernetes מזהה שחסר Pod ויוצר חדש במקומו.

כדי ש־`stats` יחזור אוטומטית במקרה של מחיקה, ניצור עבורו **Deployment** עם `replicas: 1`, כך ש־Kubernetes ידאג שתמיד יהיה Pod אחד פעיל.