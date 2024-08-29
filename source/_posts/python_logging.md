---
title: Python现成的日志函数
categories: 编程语言
mathjax: true
---



```python
def get_logger(level=logging.DEBUG):
	# Block matplotlib logging
	logging.getLogger('matplotlib').setLevel(logging.CRITICAL)

	target_logger = logging.getLogger()
	target_logger.setLevel(level)

	# Screen Handler
	stream_handler = logging.StreamHandler()
	stream_handler.setLevel(level)

	# Logging format
	logging_format = colorlog.ColoredFormatter(
		'%(log_color)s[%(levelname)s]: |%(asctime)s| |%(filename)s:%(lineno)s|: %(message)s',
		datefmt='%Y-%m-%d %H:%M:%S',
		log_colors={
			'DEBUG': 'cyan',
			'INFO': 'green',
			'WARNING': 'yellow',
			'ERROR': 'red',
			'CRITICAL': 'red,bg_white',
		}
	)
	stream_handler.setFormatter(logging_format)
	for handler in target_logger.handlers:
		target_logger.removeHandler(handler)
	target_logger.addHandler(stream_handler)
	return target_logger

```

