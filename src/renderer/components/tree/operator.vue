

<script>
export default {
  render(h) {
    return h();
  },
  methods: {
    remove(node, data) {
      if (node.level === 1) {
        //host，此时为删除数据
        this.$confirm("确认删除连接-" + node.data.label + "?", "删除链接", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            if (data.client) {
              console.log("client end");
              //关闭连接 使用end不会触发end事件
              data.client.end(true);
              // data.client.quit();
            }
            this.$store
              .dispatch("removeHost", data.label)
              .then(value => {
                this.$message("删除成功");
                this.$emit("key-remove", node.parent, {
                  label: data.label,
                  key: data.label
                });
              })
              .catch(value => {
                throw value;
                console.log(value);
                this.$message.error("删除失败");
              });
          })
          .catch(() => {});
      } else if (node.level === 2) {
        this.$confirm("确认删除数据-" + data.label + "?", "删除", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          let client = data.client || data.parent.client;
          if (!client) {
            //删除key
            client = this.createClient(data.parent);
          }
          let temp = client;
          if (client.then) {
            temp = client;
          } else {
            temp = client.delAsync(data.label);
          }
          temp
            .then(() => {
              //删除key
              this.$message("删除成功");
              console.log(node);
              console.log(data);
              this.$refs.tree.remove(node);
              this.$emit("key-remove", node.parent, {
                label: data.parent.label,
                key: data.label
              });
            })
            .catch(err => {
              console.log("删除失败");
              console.log(err);
              this.$message.error(`${data.label}删除失败`);
            });

          // let index  = node.parent.data.children.findIndex((d)=> d.label === data.label );
          // node.parent.data.children.splic(index,1);
          //通知上级已关闭对应的tab
        });
      }
    },

    refresh(node, data) {
      if (node.level === 1) {
        //host
        let client = Promise.resolve(data.index);
        if (!data.index) {
          client = this.createClient(data);
        }
        client.then(index => {
          this.$redis.keysAsync(index, "*").then(value => {
            let keys = [];
            value.forEach(e => {
              keys.push({
                label: e,
                host: data.host,
                port: data.port,
                left: true,
                isLeaf: true,
                index: index
              });
            });
            var theChildren = node.childNodes;
            theChildren.splice(0, theChildren.length);
            //FIXME  新刷新出来的key带有下拉按钮，不被认为是叶子节点

            node.doCreateChildren(keys);
            //TODO 优化去除叶子节点的逻辑，这个实现方案过于野蛮
            node.childNodes.forEach(e => {
              e.isLeaf = true;
            });
          });
        });
      } else if (node.level === 1) {
        //key
      }
    },
    createClient(data) {
      return this.$redis.init(data.host, data.port, data.password, data.index);
      // .then((key,client)=>{

      // })

      // return new Promise((resolve, reject) => {

      //   console.log(options);
      //   if (data.index !== 0) {
      //     options.db = data.index;
      //   }
      //   if (data.password !== "") {
      //     options.password = data.password;
      //   }

      //   let client = this.$redis.redis
      //     .createClient(options)
      //     .on("error", err => {
      //       console.log("event error");
      //       this.$message.error(`${data.label}连接出现错误`);
      //       data.client = null;
      //       reject(err);
      //     })
      //     .on("ready", () => {
      //       resolve(client);
      //     })
      //     .on("end", () => {
      //       console.log("redis end");
      //       data.client = null;
      //       this.$notify({
      //         duration: 0,
      //         message: `${data.label}连接已断开`,
      //         position: "bottom-right"
      //       });
      //       reject();
      //     });
      //   data.client = client;
      // });
    }
  }
};
</script>
