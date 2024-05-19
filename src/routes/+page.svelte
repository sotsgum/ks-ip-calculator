<script lang="ts">
	import { IPv4Mask, IPv4CidrRange } from 'ip-num';

	let invalidIp = $state(false);
	let invalidMask = $state(false);

	let ip = $state('182.107.165.1');
	let mask = $state('255.255.255.248');

	let main = $state({
		ip: {
			binary: ''
		},
		mask: {
			prefix: '',
			binary: ''
		},
		network: {
			decimal: '',
			binary: ''
		},
		firstHost: {
			decimal: '',
			binary: ''
		},
		lastHost: {
			decimal: '',
			binary: ''
		},
		broadcast: {
			decimal: '',
			binary: ''
		},
		hosts: {
			decimal: '',
			binary: ''
		}
	});

	let firstSubnet = $state({
		network: {
			decimal: '',
			binary: ''
		},
		firstHost: {
			decimal: '',
			binary: ''
		},
		lastHost: {
			decimal: '',
			binary: ''
		},
		broadcast: {
			decimal: '',
			binary: ''
		},
		hosts: {
			decimal: '',
			binary: ''
		}
	});

	let secondSubnet = $state({
		network: {
			decimal: '',
			binary: ''
		},
		firstHost: {
			decimal: '',
			binary: ''
		},
		lastHost: {
			decimal: '',
			binary: ''
		},
		broadcast: {
			decimal: '',
			binary: ''
		},
		hosts: {
			decimal: '',
			binary: ''
		}
	});

	function update() {
		let maskPrefix: number;
		try {
			maskPrefix = new IPv4Mask(mask).prefix;
		} catch (error) {
			invalidMask = true;
			return;
		}
		invalidMask = false;

		let range: IPv4CidrRange;
		try {
			range = IPv4CidrRange.fromCidr(`${ip}/${maskPrefix}`);
		} catch (error) {
			invalidIp = true;
			return;
		}
		invalidIp = false;

		main.ip.binary = decimalToBinary(ip);
		main.mask.prefix = `/${maskPrefix}`;
		main.mask.binary = decimalToBinary(mask);

		main.network.decimal = range.getFirst().toString();
		main.network.binary = decimalToBinary(main.network.decimal);

		main.firstHost.decimal = range.getFirst().nextIPNumber().toString();
		main.firstHost.binary = decimalToBinary(main.firstHost.decimal);

		main.lastHost.decimal = range.getLast().previousIPNumber().toString();
		main.lastHost.binary = decimalToBinary(main.lastHost.decimal);

		main.broadcast.decimal = range.getLast().toString();
		main.broadcast.binary = decimalToBinary(main.broadcast.decimal);

		const hosts = Number(range.getSize()) - 2;
		main.hosts.decimal = `2^(32-${maskPrefix}) - 2 = ${hosts}`;
		main.hosts.binary = hosts.toString(2);

		const lastOfFirst = updateFirstSubnet(main.network.decimal, maskPrefix);
		updateSecondSubnet(lastOfFirst.nextIPNumber().toString(), maskPrefix);
	}

	function updateFirstSubnet(networkAddress: string, prefix: number) {
		const maskPrefix = prefix + 1;
		const range = IPv4CidrRange.fromCidr(`${networkAddress}/${maskPrefix}`);

		firstSubnet.network.decimal = range.getFirst().toString();
		firstSubnet.network.binary = decimalToBinary(firstSubnet.network.decimal);

		firstSubnet.firstHost.decimal = range.getFirst().nextIPNumber().toString();
		firstSubnet.firstHost.binary = decimalToBinary(firstSubnet.firstHost.decimal);

		firstSubnet.lastHost.decimal = range.getLast().previousIPNumber().toString();
		firstSubnet.lastHost.binary = decimalToBinary(firstSubnet.lastHost.decimal);

		const last = range.getLast();
		firstSubnet.broadcast.decimal = last.toString();
		firstSubnet.broadcast.binary = decimalToBinary(firstSubnet.broadcast.decimal);

		const hosts = Number(range.getSize()) - 2;
		firstSubnet.hosts.decimal = `2^${32 - maskPrefix} - 2 =${hosts}`;
		firstSubnet.hosts.binary = hosts.toString(2);

		return last;
	}

	function updateSecondSubnet(networkAddress: string, prefix: number) {
		const maskPrefix = prefix + 1;
		const range = IPv4CidrRange.fromCidr(`${networkAddress}/${maskPrefix}`);

		secondSubnet.network.decimal = range.getFirst().toString();
		secondSubnet.network.binary = decimalToBinary(secondSubnet.network.decimal);

		secondSubnet.firstHost.decimal = range.getFirst().nextIPNumber().toString();
		secondSubnet.firstHost.binary = decimalToBinary(secondSubnet.firstHost.decimal);

		secondSubnet.lastHost.decimal = range.getLast().previousIPNumber().toString();
		secondSubnet.lastHost.binary = decimalToBinary(secondSubnet.lastHost.decimal);

		secondSubnet.broadcast.decimal = range.getLast().toString();
		secondSubnet.broadcast.binary = decimalToBinary(secondSubnet.broadcast.decimal);

		const hosts = Number(range.getSize()) - 2;
		secondSubnet.hosts.decimal = `${hosts}`;
		secondSubnet.hosts.binary = hosts.toString(2);
	}

	function decimalToBinary(decimal: string): string {
		const parts = decimal.split('.');
		const binaryParts: string[] = [];
		for (const part of parts) {
			const binary = parseInt(part).toString(2).padStart(8, '0');
			binaryParts.push(binary);
		}
		return binaryParts.join('.');
	}

	$inspect(invalidIp);
</script>

<main class="p-4 md:p-8">
	<h1 class="mb-8 scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl">
		Калькулятор IP-адресов (задание по КС)
	</h1>
	<h3>
		Примечание: чтобы сэкономить время, лучше пишите сначала обычные айпишники, а потом двоичные
	</h3>
	<table class="mb-4 w-full">
		<thead>
			<tr>
				<th>Имя</th>
				<th>Значение 10 СС</th>
				<th>Бинарное значение (2 СС)</th>
			</tr>
		</thead>
		<tbody>
			<tr>
				<td class="font-bold">IP адрес</td>
				<td>
					<input
						type="text"
						placeholder="182.107.165.1"
						bind:value={ip}
						aria-invalid={invalidIp}
						onchange={update}
					/>
				</td>
				<td>{main.ip.binary}</td>
			</tr>
			<tr>
				<td class="font-bold">Маска</td>
				<td>
					<input
						type="text"
						placeholder="255.255.255.248"
						bind:value={mask}
						aria-invalid={invalidMask}
						onchange={update}
					/>
					<span>{main.mask.prefix}</span>
				</td>
				<td>{main.mask.binary}</td>
			</tr>
			<tr>
				<td class="font-bold">Адрес сети</td>
				<td>{main.network.decimal}</td>
				<td>{main.network.binary}</td>
			</tr>
			<tr>
				<td class="font-bold">IP адреса первого хоста</td>
				<td>{main.firstHost.decimal}</td>
				<td>{main.firstHost.binary}</td>
			</tr>
			<tr>
				<td class="font-bold">IP адреса последнего хоста</td>
				<td>{main.lastHost.decimal}</td>
				<td>{main.lastHost.binary}</td>
			</tr>
			<tr>
				<td class="font-bold">Широковещательный адрес</td>
				<td>{main.broadcast.decimal}</td>
				<td>{main.broadcast.binary}</td>
			</tr>
			<tr>
				<td class="font-bold">Количество хостов</td>
				<td>{main.hosts.decimal}</td>
				<td>{main.hosts.binary}</td>
			</tr>
		</tbody>
	</table>
	{#each [firstSubnet, secondSubnet] as subnet, index}
		<h3>{index + 1} подсеть</h3>
		<table class="mb-4 w-full">
			<thead>
				<tr>
					<th>Имя</th>
					<th>Значение 10 СС</th>
					<th>Бинарное значение (2 СС)</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<td class="font-bold">Сеть:</td>
					<td>{subnet.network.decimal}</td>
					<td>{subnet.network.binary}</td>
				</tr>
				<tr>
					<td class="font-bold">Хост (min):</td>
					<td>{subnet.firstHost.decimal}</td>
					<td>{subnet.firstHost.binary}</td>
				</tr>
				<tr>
					<td class="font-bold">Хост (max):</td>
					<td>{subnet.lastHost.decimal}</td>
					<td>{subnet.lastHost.binary}</td>
				</tr>
				<tr>
					<td class="font-bold">Broadcast:</td>
					<td>{subnet.broadcast.decimal}</td>
					<td>{subnet.broadcast.binary}</td>
				</tr>
				<tr>
					<td class="font-bold">Кол-во хостов</td>
					<td>{subnet.hosts.decimal}</td>
					<td>{subnet.hosts.binary}</td>
				</tr>
			</tbody>
		</table>
	{/each}
</main>

<style lang="postcss">
	h3 {
		@apply mb-4 scroll-m-20 text-2xl font-semibold tracking-tight;
	}

	thead tr {
		@apply m-0 border-t p-0;
	}

	thead th {
		@apply border px-4 py-2 text-left font-bold [&[align=center]]:text-center [&[align=right]]:text-right;
	}

	tbody tr {
		@apply m-0 border-t p-0;
	}

	tbody td {
		@apply border px-4 py-2 text-left [&[align=center]]:text-center [&[align=right]]:text-right;
	}

	input {
		@apply w-36 rounded-md border-[1px] border-solid border-gray-500 px-2 py-1;
	}

	input[aria-invalid='true'] {
		@apply border-red-500 bg-red-200;
	}
</style>
