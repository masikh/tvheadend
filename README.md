# tvheadend
Get channel url, name and icons from tvheadend server

# Python code

    #!/usr/bin/env python3
    import json
    import re
    import requests

    class TVHeadend:
        def __init__(self, tvheadend_server):
            self.tvheadend_server = f'http://{tvheadend_server}:9981'
            self.regexes = [re.compile("^$"), re.compile("\W"), re.compile("astra \d/\d+\w/\W\w+:\d+\W")]

        def regex_match(self, string):
            if any(regex.match(string) for regex in self.regexes):
                return True
            return False

        def set_channel(self, channel_dict):
            key = channel_dict['uuid']
            name = channel_dict['name']
            icon = channel_dict['icon_public_url']
            icon_url = f'{self.tvheadend_server}/{icon}'
            channel = f'{self.tvheadend_server}/stream/channel/{key}'
            return {'channel': channel, 'name': name, 'icon': icon_url}

        def get_channels(self):
            url = '/api/channel/grid?limit=40000'
            query = '%s%s' % (
                self.tvheadend_server,
                url,
            )
            response = requests.get(query)
            if response.status_code != 200:
                print(f'Caught error retrieving channel list: {response.text}')
                return {}

            data = json.loads(response.text, strict=False)
            channels = [self.set_channel(x) for x in data['entries'] if not self.regex_match(x['name'])]
            return channels

    if __name__ == '__main__':
        tvh = TVHeadend('172.16.100.9')
        tvh.get_channels()
