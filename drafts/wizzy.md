
Faster way to install grafana dashboard is using [`wizzy`](https://grafana-wizzy.com/home/) (`npm i -g wizzy` if you don't have it yet)

```console
 ~ $ wizzy init
✔ conf directory created.
✔ conf file created.
✔ wizzy successfully initialized.

~ $  wizzy set grafana url http://localhost:3000/
✔ grafana:url updated successfully.
✔ conf file saved.

~ $ wizzy set grafana username admin    
✔ grafana:username updated successfully.
✔ conf file saved.

~ $ wizzy set grafana password mypassword    
✔ grafana:password updated successfully.
✔ conf file saved.
```