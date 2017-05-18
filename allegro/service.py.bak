import imp
from . import rpc

class BaseService(rpc.RpcServer):
    def __init__(self, queue, consumer, module, path):
        super(BaseService, self).__init__()

        self.queue = queue
        self._init_rpc("localhost", self.queue)
        try:
            file, pathname, desc = imp.find_module(module,[path])
            self.moduleobj = imp.load_module(consumer, file, pathname, desc)
            self.consumer = "self.moduleobj.%s" % consumer
        except Exception as e:
            raise

    def handler_request(self, message):
        """handle a specific request from amqp::templates
        """
        body = dict()
        optype = message.pop("optype")
        consumer = eval(self.consumer)()
        for content in message.keys():
            if message[content] != {} and message[content] != None:
                body[content] = message[content]
        return self._request_handler(consumer, optype, body)

