from sanic import Sanic
from sanic.response import json
from sanic.response import text

import os
import configparser
import signal

from .controller import BaseView
from .rpc import RpcClient
import logging

from asyncio import get_event_loop

from .service import BaseService

class Allegro(object):
    def __init__(self, name):
        self.log = logging.getLogger('allegro')


        formatter = logging.Formatter(
                    "%(asctime)s: %(levelname)s: %(message)s")
        handler = logging.StreamHandler()
        handler.setFormatter(formatter)
        self.log.addHandler(handler)
        self.log.setLevel(logging.INFO)

        self.app = Sanic(name)

    
    def initialize(self, config_path):
         
        try:
            self.cf = configparser.ConfigParser()
            self.cf.read(config_path)
         
            # SERVER
            self.host = self.cf.get("default", "bind_host")
            self.port = self.cf.getint("default", "bind_port")
            self.wokers = self.cf.getint("default", "api_worker")
            self.consumer_path = self.cf.get("default", "consumer_path")
            self.pid_path = self.cf.get("default", "pid_path")
            #open(self.pid_path,"w+").close()

        except Exception as e:
            self.log.exception("Error ocurred when trying to the config file Info: %s" % str(e))
            raise

    def init_route(self):
        services = self.cf.get("service", "keys").split(',')
        for service in services:
            rpc_client = RpcClient()
            queue = self.cf.get(service, "queue") 
            uri = self.cf.get(service, "uri")
            params =  self.cf.getint(service, "params")
            self.app.add_route(BaseView.as_view(queue, rpc_client, params), uri)

    def start(self):
        try:
            self.init_route()
            print(self.pid_path)
            with open(self.pid_path, "a") as f:
                f.write(str(os.getpid())+"\n")

            #self.app.run(host=self.host, port=self.port, workers=self.wokers)
            self.log.info("Starting Consumer service...")
            services = self.cf.get("service", "keys").split(',')
            for service in services:
                processList = []
                workers = self.cf.getint(service, "workers")
                consumer = self.cf.get(service, "consumer")
                queue = self.cf.get(service, "queue")
                module = self.cf.get(service, "module")
                for i in range(workers) :
                    try:
                        p = BaseService(queue, consumer, module, self.consumer_path)
                    except Exception as e:
                        self.log.exception("Failed to import %s module from %s" % (self.consumer))
                        raise
                    processList.append(p)
                    p.start()
                for p in processList:
                    with open(self.pid_path, "a") as f:
                        f.write(str(p.pid)+"\n")
            self.log.info("Stiiiarting API service...")

            self.app.run(host=self.host, port=self.port, workers=self.wokers, after_start=self.save_pid)


        except Exception as e:
            self.log.exception("Error ocurred when trying to the config file Info: %s" % str(e))
            raise
    def save_pid(self, app, loop):
        with open(self.pid_path, "a") as f:
            f.write(str(os.getpid())+"\n")

    def stop(self):
        try:
            self.log.info("Staring to termina the processes...")
            with open(self.pid_path, "r") as f:
                processes = f.readlines()
                for p in processes:
                    try:
                        a = os.kill(int(p[:-1]), signal.SIGKILL)
                    except Exception as e:
                        self.log.error(e)
            open(self.pid_path, "w+")
            self.log.info("All the processes are terminated.")
        except Exception as e:
            log.exception("Error ocurred when trying to kill the processes Info: %s" % str(e)) 
            raise

